.. _problems-and-migration-plan-2:

Problem And Migration Plan 2
============================

.. rst-class:: lead

This document outlines the **current architectural issues** in the ESP32 dashboard firmware
and provides a **migration strategy** to implement a professional FreeRTOS-based event-driven
architecture with proper state management and task coordination.

.. rubric:: 1. Current Architecture Assessment

My current main.c follows a sequential execution pattern that I need to refactor:

- Initialize Wi-Fi
- Wait for fixed delays using vTaskDelay()
- Connect to network
- Start HTTP server

This approach is functional but fragile. It doesn't handle asynchronous events properly and relies on timing assumptions that may not hold in production environments.

.. rubric:: 2. Transition to Event-Driven Architecture

Adopt FreeRTOS Event-Driven Pattern
------------------------------------

I need to restructure my application using ESP-IDF's event loop system with FreeRTOS primitives:

1. **Initialize all components** during app_main()
2. **Register event handlers** for Wi-Fi and IP events
3. **React to system events** such as:
   - WIFI_EVENT_STA_START
   - WIFI_EVENT_STA_CONNECTED
   - IP_EVENT_STA_GOT_IP
   - WIFI_EVENT_STA_DISCONNECTED
   - IP_EVENT_STA_LOST_IP

This pattern aligns with ESP-IDF best practices and leverages the built-in event loop infrastructure.

Replace Blocking Delays with Event-Driven Triggers
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Currently, I use vTaskDelay() to wait for Wi-Fi connection, which is inefficient. Instead:

- **Start HTTP server** on IP_EVENT_STA_GOT_IP event
- **Stop HTTP server** on WIFI_EVENT_STA_DISCONNECTED event
- **Use event flags or queues** to signal state changes between tasks

This eliminates arbitrary timing dependencies and makes the system responsive to actual state changes.

.. rubric:: 3. Modular Architecture Design

Separation of Concerns
----------------------

I'll enforce strict module boundaries following FreeRTOS best practices:

**wifi_api.c** - Wi-Fi Management Module
  - Wi-Fi initialization and configuration
  - Connection management (STA/AP mode switching)
  - Event posting to system event bus
  - No HTTP or application logic

**http_json_helper.c** - HTTP Server Module
  - HTTP server lifecycle management
  - JSON request/response handling
  - REST API endpoint registration
  - Subscribes to Wi-Fi events to start/stop server

**main.c** - Application Controller
  - Component initialization sequence
  - Event handler registration
  - State machine coordination
  - FreeRTOS task creation

This separation ensures each module has a single responsibility and can be tested independently.

.. rubric:: 4. State Machine Implementation

Define Wi-Fi State Enumeration
-------------------------------

Instead of using a simple g_wifi_mode flag, I'll implement a proper state machine:

.. code-block:: c

   typedef enum {
       WIFI_STATE_IDLE = 0,           // Initial state, not started
       WIFI_STATE_CONNECTING,         // STA mode, attempting connection
       WIFI_STATE_CONNECTED,          // STA mode, got IP address
       WIFI_STATE_AP_MODE,            // AP mode, accepting clients
       WIFI_STATE_ERROR,              // Error state, requires recovery
       WIFI_STATE_DISCONNECTED        // STA mode, lost connection
   } wifi_state_t;

State Transition Logic
----------------------

I'll track state transitions explicitly:

- **IDLE → CONNECTING:** When wifi_init_sta() is called
- **CONNECTING → CONNECTED:** On IP_EVENT_STA_GOT_IP
- **CONNECTED → DISCONNECTED:** On WIFI_EVENT_STA_DISCONNECTED
- **DISCONNECTED → CONNECTING:** Auto-reconnect attempt
- **Any → ERROR:** On authentication failure or timeout
- **ERROR → IDLE:** After reset or manual intervention

This makes the system behavior predictable and debuggable.

.. rubric:: 5. FreeRTOS Task Architecture

Worker Task Design
------------------

For non-blocking operations, I'll create dedicated FreeRTOS tasks:

**Wi-Fi Manager Task**
  - Priority: 5 (medium priority)
  - Stack size: 4096 bytes
  - Handles connection retry logic
  - Posts events to system event bus

**HTTP Server Task**
  - Priority: 4 (lower than Wi-Fi)
  - Stack size: 8192 bytes
  - Created only when IP is obtained
  - Deleted on disconnection to free resources

**Event Processing Task** (optional)
  - Priority: 6 (high priority for time-critical events)
  - Processes events from a FreeRTOS queue
  - Decouples event reception from event handling

Queue-Based Communication
--------------------------

I'll use FreeRTOS queues for inter-task communication:

.. code-block:: c

   // Event queue for passing events between ISR context and tasks
   QueueHandle_t wifi_event_queue;

   // In event handler (runs in system event task context)
   void wifi_event_handler(void* arg, esp_event_base_t event_base,
                           int32_t event_id, void* event_data) {
       // Push event to queue for worker task to process
       xQueueSend(wifi_event_queue, &event_id, 0);
   }

This prevents blocking the system event loop task.

.. rubric:: 6. Error Handling Strategy

Explicit Error Propagation
--------------------------

I'll implement comprehensive error handling:

1. **Log with context:** Use ESP_LOGE() with descriptive TAG and error codes
2. **Return ESP_FAIL:** Functions return error codes instead of void
3. **State recovery:** On error, transition to WIFI_STATE_ERROR and attempt recovery
4. **Watchdog integration:** Register tasks with TWDT to detect hangs

Example pattern:

.. code-block:: c

   esp_err_t wifi_start_http_server(void) {
       if (wifi_state != WIFI_STATE_CONNECTED) {
           ESP_LOGE(TAG, "Cannot start HTTP server: Wi-Fi not connected");
           return ESP_ERR_INVALID_STATE;
       }

       esp_err_t ret = http_server_start();
       if (ret != ESP_OK) {
           ESP_LOGE(TAG, "HTTP server failed to start: %s", esp_err_to_name(ret));
           // Post error event for system-wide notification
           event_bus_post(APP_EVENT_HTTP_SERVER_ERROR);
           return ret;
       }

       return ESP_OK;
   }

.. rubric:: 7. Production Best Practices

Avoid Blocking in Event Handlers
---------------------------------

**Critical Rule:** Event handlers must be non-blocking and execute quickly (< 100ms).

If I need to perform heavy operations (NVS write, HTTP request, file I/O):

1. **In event handler:** Push data to FreeRTOS queue
2. **In worker task:** Block on queue receive and process data
3. **Use notifications:** Alternative to queues for simple signaling

Custom Event Loop (Advanced)
-----------------------------

For complex applications, I'll create a dedicated event loop:

.. code-block:: c

   esp_event_loop_args_t loop_args = {
       .queue_size = 32,
       .task_name = "app_event_loop",
       .task_priority = 5,
       .task_stack_size = 4096,
       .task_core_id = 0
   };

   esp_event_loop_create(&loop_args, &app_event_loop_handle);

This isolates application events from ESP-IDF system events.

.. rubric:: 8. Implementation Roadmap

Phase 1: Event Bus Foundation
------------------------------

1. Define event IDs in app_event_bus.h
2. Implement event_bus_post() wrapper functions
3. Create Wi-Fi event handler skeleton
4. Test event posting and receiving

Phase 2: State Machine Integration
-----------------------------------

1. Implement wifi_state_t enum and state tracking variable
2. Add state transition logging
3. Create state validation functions
4. Implement error recovery logic

Phase 3: HTTP Server Lifecycle
-------------------------------

1. Subscribe http_json_helper to Wi-Fi events
2. Start HTTP server on IP_EVENT_STA_GOT_IP
3. Stop HTTP server on WIFI_EVENT_STA_DISCONNECTED
4. Add graceful shutdown with connection draining

Phase 4: Testing and Validation
--------------------------------

1. Test Wi-Fi reconnection scenarios
2. Verify HTTP server availability matches network state
3. Monitor task stack usage with ESP-IDF monitor
4. Enable TWDT and verify no watchdog resets
5. Load test with multiple concurrent HTTP requests

Expected Outcomes
-----------------

Implementing this architecture will result in:

- **Responsive system:** HTTP server starts within 100ms of IP acquisition
- **Resource efficient:** HTTP server task only exists when network is available
- **Maintainable code:** Clear separation of concerns, easy to debug
- **Production-ready:** Proper error handling, watchdog integration, state tracking
- **Scalable:** Easy to add new features (MQTT, OTA updates, etc.) via event subscriptions