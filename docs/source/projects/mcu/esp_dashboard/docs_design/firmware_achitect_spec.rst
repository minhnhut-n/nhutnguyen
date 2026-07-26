.. _firmware-architecture-specification:

Firmware Architecture Specification
====================================

.. rst-class:: lead

Tài liệu này mô tả kiến trúc tổng thể được đề xuất cho các dự án **ESP32** sử dụng **ESP-IDF** với mục tiêu xây dựng firmware có khả năng mở rộng, dễ bảo trì và tái sử dụng trong nhiều năm phát triển sản phẩm.

--------------------------------------------------------------------------------

.. rubric:: Giới thiệu

Trong giai đoạn đầu của một dự án firmware, việc các module gọi trực tiếp function của nhau thường **không phải là vấn đề**.

.. code-block:: text

    HTTP ──→ Storage
    Storage ──→ WiFi
    WiFi ──→ WebSocket

Khi project còn nhỏ, cách làm này giúp phát triển nhanh.

Tuy nhiên, khi sản phẩm bắt đầu bổ sung nhiều tính năng:

.. list-table:: Các tính năng phổ biến trên ESP32
   :header-rows: 1
   :widths: 20 20 20 20 20

   * - HTTP Server
     - WebSocket
     - UART
     - BLE
     - MQTT
   * - OTA
     - FFT
     - NVS
     - Sensor
     - Cloud

thì kiến trúc ban đầu sẽ dần **bộc lộ các vấn đề** về khả năng mở rộng.

.. admonition:: Vấn đề cốt lõi
   :class: warning

   Cần chuyển từ mô hình **"function call"** sang một kiến trúc có **tính module hóa cao hơn**.

--------------------------------------------------------------------------------

.. rubric:: Mục tiêu của kiến trúc

Kiến trúc mới hướng đến các mục tiêu sau:

.. list-table:: Mục tiêu thiết kế
   :header-rows: 1
   :widths: 25 75

   * - Mục tiêu
     - Ý nghĩa
   * - **High Cohesion**
     - Mỗi module chỉ làm đúng một việc
   * - **Low Coupling**
     - Giảm phụ thuộc giữa các module
   * - **Reusable**
     - Có thể tái sử dụng giữa các dự án
   * - **Portable**
     - Dễ dàng chuyển đổi nền tảng
   * - **Maintainable**
     - Dễ bảo trì và nâng cấp
   * - **Testable**
     - Có thể viết unit test cho từng module
   * - **Scalable**
     - Có thể mở rộng khi thêm tính năng mới

Đây cũng là các tiêu chí để phân chia rõ các **tầng của hệ thống**.

.. code-block:: text
   :caption: Kiến trúc phân tầng

     ┌─────────────────────────────┐
     │         Application         │  ← Điều phối toàn bộ hệ thống
     ├─────────────────────────────┤
     │           Service           │  ← Các thành phần nghiệp vụ
     ├─────────────────────────────┤
     │          Middleware         │  ← Các thành phần dùng chung
     ├─────────────────────────────┤
     │           Driver            │  ← Driver của thiết bị
     ├─────────────────────────────┤
     │             HAL             │  ← Giao tiếp trực tiếp với ESP-IDF
     └─────────────────────────────┘

Application
-----------

Điều phối toàn bộ hệ thống.

* ``app_main()``
* ``boot sequence``
* ``startup logic``

Service
-------

Bao gồm các thành phần nghiệp vụ:

* HTTP
* MQTT
* WiFi
* WebSocket
* OTA
* FFT

Middleware
----------

Các thành phần dùng chung:

* Event Bus
* Config Manager
* Storage API
* Logger
* Message Queue

Driver
------

Driver của thiết bị:

* UART
* SPI
* I2C
* GPIO
* ADC
* PWM

HAL
---

Lớp giao tiếp trực tiếp với ESP-IDF.

* ``esp_wifi``
* ``nvs_flash``
* ``esp_http_server``

--------------------------------------------------------------------------------

.. rubric:: Nguyên tắc Dependency

Dependency chỉ được phép theo **một chiều**.

.. code-block:: text
   :caption: Chiều dependency hợp lệ

     Application
          │
          ▼
       Service
          │
          ▼
     Middleware
          │
          ▼
       Driver

.. danger:: Không được phép

   .. code-block:: text

         Driver           Storage
            │                │
            ▼                ▼
          HTTP             HTTP

Điều này giúp loại bỏ **Circular Dependency**.

--------------------------------------------------------------------------------

.. rubric:: Module hóa

Mỗi module phải có **một trách nhiệm duy nhất**.

.. list-table:: Ví dụ - WiFi Module
   :header-rows: 1
   :widths: 50 50

   * - ✅ Quản lý
     - ❌ Không xử lý
   * - ``connect``
     - HTTP
   * - ``disconnect``
     - MQTT
   * - ``reconnect``
     - Storage
   * - ``scan``
     -

--------------------------------------------------------------------------------

.. rubric:: Module Lifecycle

Tất cả module nên **thống nhất API**.

.. code-block:: c
   :caption: API lifecycle chuẩn

     module_init()
     module_start()
     module_stop()
     module_reset()
     module_deinit()

Điều này giúp:

* **Quản lý** dễ hơn
* **Restart service** dễ dàng
* **Test** đơn giản hơn

--------------------------------------------------------------------------------

.. rubric:: FreeRTOS là nền tảng

ESP-IDF đã tích hợp FreeRTOS. Do đó mỗi service nên chạy dưới dạng **Task độc lập**.

.. code-block:: text
   :caption: Các Task trong hệ thống

     ┌─────────────┬─────────────┬─────────────┬─────────────┐
     │  WiFi Task  │  HTTP Task  │Storage Task │  UART Task  │
     ├─────────────┼─────────────┼─────────────┼─────────────┤
     │  FFT Task   │  OTA Task   │WebSocket Tk │  BLE Task   │
     └─────────────┴─────────────┴─────────────┴─────────────┘

.. note::

   Mỗi Task chỉ xử lý **đúng nhiệm vụ của mình**.

--------------------------------------------------------------------------------

.. rubric:: Event Bus

Đây là **trung tâm của kiến trúc**.

Module **không gọi trực tiếp** module khác.

.. code-block:: text
   :caption: Cơ chế Event Bus

     HTTP                        WiFi
        │                          ▲
        │    ┌───────────────┐     │
        └───→│   Publish     │─────┘
             │               │
             │   Event Bus   │
        ┌───→│               │──────┐
        │    └───────────────┘      │
        │                           │
     MQTT                          OTA

HTTP **không biết** WiFi tồn tại.
WiFi cũng **không biết** HTTP tồn tại.

.. rubric:: Event chỉ mang ý nghĩa Notification

.. code-block:: text
   :caption: Các event mẫu

     WIFI_CONNECTED
     MQTT_CONNECTED
     OTA_FINISHED
     HTTP_REQUEST_RECEIVED
     UART_TIMEOUT

.. important::

   Không dùng Event để **truyền dữ liệu lớn**.

--------------------------------------------------------------------------------

.. rubric:: Queue

Queue dùng để **truyền dữ liệu**.

.. code-block:: text
   :caption: Ví dụ truyền dữ liệu qua Queue

     ┌──────┐     ┌─────────┐     ┌─────┐
     │ UART │────→│  Frame  │────→│ FFT │
     └──────┘     └─────────┘     └─────┘

     ┌──────┐     ┌──────────────┐     ┌─────────┐
     │ HTTP │────→│SSID+PASSWORD │────→│ Storage │
     └──────┘     └──────────────┘     └─────────┘

Các cơ chế queue nên sử dụng:

* FreeRTOS Queue
* Ring Buffer
* Stream Buffer

--------------------------------------------------------------------------------

.. rubric:: Event + Queue

.. admonition:: Quy ước quan trọng
   :class: tip

   * **Event** → thông báo
   * **Queue** → truyền dữ liệu

.. code-block:: text
   :caption: Luồng xử lý kết hợp Event và Queue

     HTTP nhận request
           │
           ▼
     Push Config vào Queue
           │
           ▼
     Publish CONFIG_UPDATED
           │
           ▼
     Storage nhận Queue
           │
           ▼
     Save NVS

--------------------------------------------------------------------------------

.. rubric:: State Machine

Các module có trạng thái nên xây dựng dưới dạng **State Machine**.

.. list-table:: Ví dụ State Machine
   :header-rows: 1
   :widths: 25 25 25

   * - WiFi
     - OTA
     - MQTT
   * - ::

           Disconnected
               │
               ▼
           Connecting
               │
               ▼
           Connected
               │
               ▼
         Internet Ready
               │
               ▼
             Lost
               │
               ▼
           Reconnect

     - ::

             Idle
               │
               ▼
          Downloading
               │
               ▼
            Verifying
               │
               ▼
           Installing
               │
               ▼
             Done

     - ::

        Disconnected
               │
               ▼
           Connecting
               │
               ▼
           Connected
               │
               ▼
          Reconnecting

State Machine giúp:

* **Tránh logic chồng chéo**
* **Dễ debug**
* **Dễ mở rộng**

--------------------------------------------------------------------------------

.. rubric:: Configuration Manager

Không để mỗi module đọc ghi NVS trực tiếp.

.. code-block:: text
   :caption: Kiến trúc Config Manager

                         ┌───────────────┐
                         │      NVS      │
                         └───────┬───────┘
                                 │
                         ┌───────▼───────┐
                         │Config Manager │
                         └───────┬───────┘
                                 │
         ┌───────────┬───────────┼───────────┬───────────┐
         │           │           │           │           │
         ▼           ▼           ▼           ▼           ▼
        WiFi        MQTT      Device      Calib-     FFT Config
                               Name       ration

.. note::

   **Storage** chỉ lưu dữ liệu. **Config Manager** quản lý cấu hình.

--------------------------------------------------------------------------------

.. rubric:: Storage Abstraction

Application **không gọi trực tiếp** API của NVS.

.. code-block:: c
   :caption: ❌ Cách làm sai

     nvs_set_str()
     nvs_get_str()

.. code-block:: c
   :caption: ✅ Cách làm đúng

     storage_save()
     storage_load()

.. hint:: Lợi ích

   Sau này đổi:

   * NVS
   * SPIFFS
   * LittleFS
   * EEPROM

   **không cần sửa Application**.

--------------------------------------------------------------------------------

.. rubric:: Service Manager

Một thành phần **điều phối việc khởi động hệ thống**.

.. code-block:: text
   :caption: Startup sequence

     ┌─────────────┐
     │    Boot     │
     └──────┬──────┘
            ▼
     ┌─────────────┐
     │   Storage   │
     └──────┬──────┘
            ▼
     ┌─────────────┐
     │ Load Config │
     └──────┬──────┘
            ▼
     ┌─────────────┐
     │    WiFi     │
     └──────┬──────┘
            ▼
     ┌─────────────┐
     │    HTTP     │
     └──────┬──────┘
            ▼
     ┌─────────────┐
     │ WebSocket   │
     └──────┬──────┘
            ▼
     ┌─────────────┐
     │    MQTT     │
     └─────────────┘

.. important::

   Toàn bộ startup sequence nằm ở **một nơi duy nhất**.

--------------------------------------------------------------------------------

.. rubric:: Logging Framework

.. warning:: Không nên dùng

   .. code-block:: c

         printf()

.. list-table:: Các mức log chuẩn
   :header-rows: 1
   :widths: 25 75

   * - Macro
     - Mục đích
   * - ``LOG_DEBUG``
     - Thông tin debug chi tiết
   * - ``LOG_INFO``
     - Thông tin vận hành bình thường
   * - ``LOG_WARN``
     - Cảnh báo, cần lưu ý
   * - ``LOG_ERROR``
     - Lỗi cần xử lý

Sau này log có thể được xuất ra:

.. list-table:: Đầu ra log
   :widths: 25 25 25 25 25

   * - UART
     - WebSocket
     - Flash
     - SD Card
     - Cloud

mà **không cần sửa code**.

--------------------------------------------------------------------------------

.. rubric:: Interface First

Module **không expose implementation**.

.. code-block:: text

     wifi.c
       │
       ▼
     wifi.h

Application chỉ biết:

.. code-block:: c

     wifi_connect()
     wifi_disconnect()

.. note::

   Application **không biết** module bên trong hoạt động thế nào.

--------------------------------------------------------------------------------

.. rubric:: Dependency Injection

Module **không tự tìm dependency**.

.. list-table:: So sánh
   :header-rows: 1
   :widths: 50 50

   * - ❌ Cách cũ
     - ✅ Cách mới
   * - ::

           WiFi
             │
             ▼
          Storage

     - ::

           wifi_init(storage_if);

Điều này giúp module:

* **Portable** — có thể chạy trên nhiều nền tảng
* **Testable** — dễ dàng mock trong unit test

--------------------------------------------------------------------------------

.. rubric:: Coding Convention

Một số quy ước nên thống nhất ngay từ đầu.

.. rubric:: Folder

.. code-block:: text

     app/
     components/
     services/
     middleware/
     drivers/
     hal/
     common/

.. rubric:: File

.. code-block:: text

     wifi_manager.c
     wifi_manager.h
     storage.c
     storage.h

.. rubric:: Event

.. code-block:: text

     EVENT_WIFI_CONNECTED
     EVENT_HTTP_REQUEST
     EVENT_OTA_DONE

.. rubric:: Queue

.. code-block:: text

     wifi_queue
     uart_queue
     storage_queue

.. rubric:: Task

.. code-block:: text

     wifi_task
     http_task
     fft_task

--------------------------------------------------------------------------------

.. rubric:: Unit Testing

.. admonition:: Chiến lược kiểm thử
   :class: seealso

   * Viết Unit Test cho **từng module**
   * Sử dụng **mock** cho các dependency bên ngoài
   * Kiểm tra **luồng Event** và **trạng thái State Machine**
   * Tích hợp vào **CI pipeline** tự động

--------------------------------------------------------------------------------

.. rubric:: Các nguyên tắc cần ghi nhớ

.. list-table:: 12 nguyên tắc vàng
   :header-rows: 1
   :widths: 5 95

   * - #
     - Nguyên tắc
   * - 1
     - Một module chỉ làm **một việc**
   * - 2
     - Không gọi trực tiếp module khác nếu có thể dùng **Event**
   * - 3
     - **Event** dùng để thông báo
   * - 4
     - **Queue** dùng để truyền dữ liệu
   * - 5
     - Không truyền payload lớn qua **Event Bus**
   * - 6
     - Không **include chéo** giữa các module
   * - 7
     - Chỉ phụ thuộc **xuống tầng dưới**
   * - 8
     - Mọi module đều có **Lifecycle thống nhất**
   * - 9
     - Storage phải được **trừu tượng hóa**
   * - 10
     - Mọi cấu hình đi qua **Config Manager**
   * - 11
     - Mọi service chạy dưới **FreeRTOS Task**
   * - 12
     - Mọi trạng thái phức tạp nên dùng **State Machine**

--------------------------------------------------------------------------------

.. rubric:: Lộ trình triển khai (tham khảo)

Việc chuyển đổi **không cần thực hiện trong một lần**. Một lộ trình khả thi là:

.. code-block:: text

   timeline
     title Lộ trình chuyển đổi kiến trúc
     Giai đoạn 1 – Chuẩn hóa cấu trúc
       : Tách thư mục theo Layer
       : Loại bỏ include chéo
       : Chuẩn hóa naming convention
       : Thống nhất lifecycle của module
     Giai đoạn 2 – Đưa FreeRTOS vào kiến trúc
       : Chuyển thành phần lớn → Task độc lập
       : Xây dựng Service Manager
       : Áp dụng State Machine
     Giai đoạn 3 – Nâng cao chất lượng
       : Viết Unit Test cho từng module
       : Tích hợp CI/CD
       : Code review kiến trúc

.. rubric:: Giai đoạn 1 – Chuẩn hóa cấu trúc

* Tách thư mục theo **Layer**
* Loại bỏ **include chéo**
* Chuẩn hóa **naming convention**
* Thống nhất **lifecycle** của module

.. rubric:: Giai đoạn 2 – Đưa FreeRTOS vào kiến trúc

* Chuyển các thành phần lớn (WiFi, HTTP, UART...) thành các **Task độc lập**
* Xây dựng **Service Manager** để quản lý vòng đời
* Áp dụng **State Machine** cho các service có nhiều trạng thái (Wi-Fi, OTA, MQTT...)

.. rubric:: Giai đoạn 3 – Nâng cao chất lượng

* Viết **Unit Test** cho từng module
* Tích hợp **CI/CD** tự động
* **Code review** kiến trúc định kỳ

--------------------------------------------------------------------------------

.. rubric:: Kết luận

.. rst-class:: conclusion

   Đối với một dự án **ESP32** sử dụng **ESP-IDF** có định hướng phát triển lâu dài, kiến trúc

   **Layered + FreeRTOS + Event Bus + Queue + Interface**

   là một **nền tảng vững chắc**. Không chỉ giúp **giảm coupling** và **tăng khả năng tái sử dụng**,
   mà còn tạo tiền đề để firmware có thể **mở rộng** thêm nhiều giao thức, nhiều thiết bị ngoại vi
   và nhiều dịch vụ trong tương lai mà **không làm mất đi tính ổn định** và **khả năng bảo trì** của hệ thống.

.. raw:: html

   <hr style="border: 1px solid #2980b9; margin: 2em 0;">
   <p style="text-align: center; color: #7f8c8d;">
   <em>Firmware Architecture Specification v1.0</em>
   </p>