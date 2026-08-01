II. Vertical Slice and Event-Driven Trên ESP32 (FreeRTOS)
=====================================================================================

Dưới đây là thiết kế chi tiết và mã nguồn minh họa việc áp dụng kết hợp Vertical Slice Architecture (Cắt dọc tính năng) và Event-Driven Architecture (Hướng sự kiện) trên vi điều khiển ESP32 sử dụng môi trường ESP-IDF / FreeRTOS.

.. rubric:: 1. Bài Toán Thực Tế: Hệ Thống Smart Environment Monitor

Hệ thống bao gồm 2 tính năng chính (2 Vertical Slices):

- **Slice 1 (Temperature Alert):** Đọc cảm biến nhiệt độ (ví dụ DHT22/BME280). Khi nhiệt độ vượt ngưỡng (> 35°C), phát tín hiệu cảnh báo.
- **Slice 2 (User Control / LED Indicator):** Nhận sự kiện bấm nút từ người dùng để đổi chế độ cảnh báo hoặc điều khiển đèn báo trạng thái.

.. rubric:: 2. Cấu Trúc Thư Mục Theo Vertical Slice Architecture

Thay vì chia thư mục dạng drivers/, services/, models/, dự án được tổ chức theo từng tính năng độc lập:

::

  main/
  ├── core/
  │   └── event_bus.h           <-- Shared Event Bus (Cross-Cutting Concern)
  ├── slices/
  │   ├── temp_monitor/         <-- Vertical Slice 1: Nhiệt độ
  │   │   ├── temp_sensor.c     (Driver Hardware)
  │   │   ├── temp_logic.c      (Business Logic)
  │   │   └── temp_slice.h      (Interface khởi tạo slice)
  │   └── user_button/          <-- Vertical Slice 2: Nút bấm & LED
  │       ├── button_driver.c   (Hardware ISR)
  │       ├── led_control.c     (Output Actuator)
  │       └── button_slice.h    (Interface khởi tạo slice)
  └── main.c                    <-- Khởi tạo bus và kích hoạt các slices

.. rubric:: 3. Thiết Kế Event Bus (Event-Driven Foundation)

Các Slices hoàn toàn không #include trực tiếp file header của nhau. Mọi tương tác diễn ra thông qua Event Loop của ESP-IDF (hoặc FreeRTOS Queue).

.. code-block:: c

  // main/core/event_bus.h
  #ifndef EVENT_BUS_H
  #define EVENT_BUS_H

  #include "esp_event.h"

  // Định nghĩa Event Base dùng chung
  ESP_EVENT_DECLARE_BASE(SYSTEM_EVENTS);

  // Các sự kiện trong hệ thống
  typedef enum {
      EVENT_TEMP_OVERHEAT,    // Slice Temperature phát ra
      EVENT_BUTTON_PRESSED    // Slice Button phát ra
  } system_event_id_t;

  #endif

---

.. rubric:: 4. Triển Khai Vertical Slice 1: Temp Monitor (Phát Event)

Slice này tự quản lý việc đọc phần cứng và logic kiểm tra ngưỡng. Nó không hề biết ai sẽ nhận tín hiệu EVENT_TEMP_OVERHEAT.

.. code-block:: c

  // main/slices/temp_monitor/temp_slice.c
  #include "freertos/FreeRTOS.h"
  #include "freertos/task.h"
  #include "esp_log.h"
  #include "core/event_bus.h"

  static const char *TAG = "TEMP_SLICE";

  static void temp_monitor_task(void *pvParameters) {
      float temperature = 25.0f;

      for (;;) {
          // 1. Đọc phần cứng (Hardware Driver Sub-layer)
          temperature += 2.5f; // Giả lập nhiệt độ tăng dần
          ESP_LOGI(TAG, "Nhiệt độ hiện tại: %.1f°C", temperature);

          // 2. Kiểm tra Business Logic
          if (temperature > 35.0f) {
              ESP_LOGW(TAG, "Cảnh báo: Nhiệt độ vượt ngưỡng! Phát Event...");
              
              // 3. Dispatch Event lên Event Bus (Event-Driven)
              esp_event_post(SYSTEM_EVENTS, EVENT_TEMP_OVERHEAT, &temperature, sizeof(temperature), portMAX_DELAY);
              
              temperature = 25.0f; // Reset giả lập
          }

          vTaskDelay(pdMS_TO_TICKS(3000)); // Đọc mỗi 3 giây
      }
  }

  void temp_slice_init(void) {
      // Chạy Task riêng trên Core 1
      xTaskCreatePinnedToCore(temp_monitor_task, "temp_task", 3072, NULL, 5, NULL, 1);
  }

.. rubric:: 5. Triển Khai Vertical Slice 2: User Control & LED (Đăng Ký Nhận Event)

Slice này lắng nghe các sự kiện từ Event Bus. Khi nhận được EVENT_TEMP_OVERHEAT hoặc EVENT_BUTTON_PRESSED, nó sẽ phản ứng mà không cần gọi trực tiếp code từ Slice Temp.

.. code-block:: c

  // main/slices/user_button/button_slice.c
  #include "freertos/FreeRTOS.h"
  #include "freertos/task.h"
  #include "esp_log.h"
  #include "core/event_bus.h"

  static const char *TAG = "BUTTON_LED_SLICE";

  // Callback xử lý khi nhận sự kiện từ Event Bus
  static void on_system_event(void* handler_args, esp_event_base_t base, int32_t id, void* event_data) {
      if (base == SYSTEM_EVENTS) {
          switch (id) {
              case EVENT_TEMP_OVERHEAT: {
                  float temp = *(float*)event_data;
                  ESP_LOGE(TAG, "[LED ACTUATOR] Bật LED Đỏ nhấp nháy khẩn cấp! (Do nhiệt độ = %.1f°C)", temp);
                  break;
              }
              case EVENT_BUTTON_PRESSED:
                  ESP_LOGI(TAG, "[LED ACTUATOR] Tắt báo động do người dùng nhấn nút Reset.");
                  break;
              default:
                  break;
          }
      }
  }

  void button_slice_init(void) {
      // Đăng ký nhận Event từ Event Bus
      esp_event_handler_instance_register(SYSTEM_EVENTS, ESP_EVENT_ANY_ID, &on_system_event, NULL, NULL);
  }

.. rubric:: 6. Luồng Thực Thi Tại main.c

.. code-block:: c

  // main/main.c
  #include "esp_event.h"
  #include "esp_log.h"
  #include "core/event_bus.h"

  // Import interface của các Slices
  #include "slices/temp_monitor/temp_slice.h"
  #include "slices/user_button/button_slice.h"

  ESP_EVENT_DEFINE_BASE(SYSTEM_EVENTS);

  void app_main(void) {
      ESP_LOGI("MAIN", "Khởi tạo hệ thống ESP32...");

      // 1. Khởi tạo Event Loop Mặc Định của ESP-IDF (System Event Bus)
      ESP_ERROR_CHECK(esp_event_loop_create_default());

      // 2. Khởi tạo độc lập từng Vertical Slice
      button_slice_init();
      temp_slice_init();

      ESP_LOGI("MAIN", "Tất cả Vertical Slices đã hoạt động độc lập!");
  }

.. rubric:: 7. Ưu Điểm Của Sự Kết Hợp Trên ESP32

**Tiêu chí** | **Lợi ích trên ESP32**
--- | ---
**Loose Coupling (Mất Phụ Thuộc)** | Nếu bạn gỡ bỏ Slice 1 (Nhiệt độ), Slice 2 (Nút/LED) vẫn biên dịch và hoạt động bình thường mà không bị lỗi thiếu hàm hay thiếu thư viện.
**Tận Dụng Dual-Core ESP32** | Mỗi Vertical Slice có thể chạy trên một Task RTOS riêng biệt (Task Temp ở Core 1, Task Radio/Network ở Core 0) mà không lo tranh chấp tài nguyên nhờ Event Bus.
**Dễ Dàng Mở Rộng** | Muốn thêm Slice 3 (Gửi tin nhắn MQTT lên Cloud)? Chỉ cần viết Slice 3 và đăng ký lắng nghe EVENT_TEMP_OVERHEAT mà không cần sửa 1 dòng code nào trong Slice 1 hay Slice 2.