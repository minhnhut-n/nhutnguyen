.. _esp32_event_driven_architecture:

===================================================================
3. Event-Driven Structure
===================================================================

.. contents:: Mục Lục Bài Viết
   :depth: 2
   :local:

---

.. _section-problem-statement:

I. Đặt Vấn Đề & Động Cơ Tái Cấu Trúc
====================================

Trong quá trình phát triển các ứng dụng IoT phức tạp trên ESP32, việc các module (Wi-Fi, HTTP Server, NVS, Sensor, LED...) được thêm mới liên tục dễ dẫn đến tình trạng **Spaghetti Dependencies** (phụ thuộc chồng chéo). 

Vấn Đề Thực Tế
--------------
* **Tight Coupling (Gắn kết chặt):** Module A phải ``#include "module_B.h"``, Module B lại ``#include "module_C.h"``. Chỉ cần sửa 1 dòng code ở module này sẽ kéo theo hàng loạt thay đổi ở các module khác.
* **Overhead & Build Time:** Thời gian biên dịch (Build time) tăng do file header bị include chồng chéo nhiều lần.
* **Khó Debug & Kiểm Thử:** Không thể Unit Test riêng lẻ từng module vì các phụ thuộc bị dính liền với nhau.

Giải Pháp: Kiến Trúc Event-Driven (Pub-Sub)
-------------------------------------------
Chuyển đổi hệ thống sang mô hình **Bắn - Lắng nghe Tín hiệu (Publish - Subscribe)** thông qua một **Event Bus** trung tâm. Các module phát triển độc lập, hoàn toàn không biết sự tồn tại của nhau, giúp tối ưu hóa luồng code và dễ dàng mở rộng tính năng.

---

.. _section-benefits-and-risks:

II. Lợi Ích & Các Rủi Ro Cần Tránh
==================================

Lợi Ích Cốt Lõi (Benefits)
--------------------------
1. **Zero Coupling:** Các module chỉ phụ thuộc duy nhất vào thư viện **Event Bus**.
2. **Khả Năng Mở Rộng Dễ Dàng:** Muốn thêm feature mới (ví dụ: Đèn LED cảnh báo khi mất Wi-Fi)? Chỉ cần tạo module LED và Subscribe tín hiệu Mất Wi-Fi. Module Wi-Fi không cần sửa dù chỉ 1 dòng code.
3. **Chống Đổ Vỡ Dây Truyền (Fault Tolerance):** Nếu module Subscriber bị lỗi hoặc bị vô hiệu hóa, Publisher vẫn bắn tín hiệu bình thường mà không gây crash hệ thống.

Rủi Ro Kỹ Thuật Cần Tránh (Pitfalls)
------------------------------------
.. warning::
   **Rủi ro Long-Holding & Task Watchdog Reset (TWDT)**
   
   Mặc định, các Handler của Subscriber được thực thi trực tiếp trên **System Event Loop Task**. Nếu trong Handler bạn thực hiện tác vụ tốn thời gian (như ``vTaskDelay()``, ghi Flash, hoặc vòng lặp chờ I/O), bạn sẽ làm kẹt toàn bộ Event Bus của hệ thống. Hậu quả là Task Watchdog Timer sẽ bị trigger và làm Reset ESP32.

   * **Quy tắc vàng:** Event Handler chỉ dùng để xử lý dữ liệu cực nhẹ (Copy flag, cập nhật biến) hoặc đẩy dữ liệu vào FreeRTOS Queue cho một Worker Task khác xử lý bất đồng bộ.

---

.. _section-five-steps-implementation:

III. Quy Trình 5 Bước Triển Khai (Chi Tiết & Code Minh Họa)
===========================================================

Hệ thống được chia làm 3 tầng giao tiếp:

* **Event Bus Component:** Định nghĩa Base, Signal IDs và API wrapper.
* **Publisher Component:** Nơi phát tín hiệu khi trạng thái thay đổi.
* **Subscriber Component:** Nơi đăng ký nhận và xử lý tín hiệu.

.. image:: https://via.placeholder.com/600x150.png?text=Publisher+--%3E+Event+Bus+--%3E+Subscriber
   :alt: Event Driven Flowchart

Bước 1: Khai Báo Dữ Liệu & Event ID
------------------------------------
.. rubric:: Vị trí code: Phía Event Bus Component (``app_event_bus.h``)

Định nghĩa các Event ID và Cấu trúc dữ liệu đi kèm để các bên giao tiếp đúng chuẩn.

.. code-block:: c

   // File: components/event_bus/include/app_event_bus.h
   #pragma once
   #include "esp_event.h"

   // 1. Khai báo Event Base (Bus ID)
   ESP_EVENT_DECLARE_BASE(APP_SYS_EVENTS);

   // 2. Danh sách các Tín hiệu (Signals)
   typedef enum {
       APP_EVENT_WIFI_CONNECTED,
       APP_EVENT_WIFI_DISCONNECTED,
       APP_EVENT_SENSOR_DATA_READY,
   } app_event_id_t;

   // 3. Data Structure đính kèm signal (nếu có)
   typedef struct {
       float temperature;
       float humidity;
   } sensor_event_data_t;

Bước 2: Định Nghĩa Event Base (Alloc Memory)
---------------------------------------------
.. rubric:: Vị trí code: Phía Event Bus Component (``app_event_bus.c``)

Khởi tạo bộ nhớ cho ``APP_SYS_EVENTS`` để tạo ra chuỗi nhận diện duy nhất cho Bus.

.. code-block:: c

   // File: components/event_bus/app_event_bus.c
   #include "app_event_bus.h"

   // Định nghĩa duy nhất 1 lần trong toàn bộ dự án
   ESP_EVENT_DEFINE_BASE(APP_SYS_EVENTS);

Bước 3: Đóng Gói API Phát Signal (Post Wrapper)
------------------------------------------------
.. rubric:: Vị trí code: Phía Event Bus Component (``app_event_bus.c``)

Cung cấp hàm sạch sẽ cho Publisher thay vì thao tác trực tiếp với API gốc của ESP-IDF.

.. code-block:: c

   // File: components/event_bus/app_event_bus.c
   #include "app_event_bus.h"

   // API bắn signal đơn thuần không đính kèm data
   esp_err_t event_bus_post(app_event_id_t event_id) {
       return esp_event_post(APP_SYS_EVENTS, event_id, NULL, 0, portMAX_DELAY);
   }

   // API bắn signal CÓ đính kèm payload data
   esp_err_t event_bus_post_data(app_event_id_t event_id, void* data, size_t data_size) {
       return esp_event_post(APP_SYS_EVENTS, event_id, data, data_size, portMAX_DELAY);
   }

Bước 4: Phát Signal Tại Publisher Module
----------------------------------------
.. rubric:: Vị trí code: Phía Publisher Component (VD: ``wifi_manager.c``)
.. note::
   Module này **chỉ include** ``app_event_bus.h``, hoàn toàn KHÔNG include bất kỳ module nhận nào.

.. code-block:: c

   // File: components/wifi_manager/wifi_manager.c
   #include "app_event_bus.h"

   void wifi_process_connected_internal(void) {
       // Logic kết nối Wi-Fi thành công...
       
       // Bắn signal lên Event Bus
       event_bus_post(APP_EVENT_WIFI_CONNECTED);
   }

Bước 5: Đăng Ký Callback & Nhận Signal
--------------------------------------
.. rubric:: Vị trí code: Phía Subscriber Component (VD: ``main.c`` hoặc ``app_controller.c``)

.. code-block:: c

   // File: main/main.c
   #include "app_event_bus.h"
   #include "esp_log.h"

   static const char *TAG = "MAIN_APP";

   // Hàm Callback xử lý khi có Signal bắn về
   static void sys_event_handler(void* handler_args, esp_event_base_t base, 
                                 int32_t id, void* event_data) 
   {
       if (base == APP_SYS_EVENTS) {
           switch (id) {
               case APP_EVENT_WIFI_CONNECTED:
                   ESP_LOGI(TAG, "Nhận signal: Wi-Fi đã kết nối!");
                   break;
                   
               case APP_EVENT_SENSOR_DATA_READY: {
                   sensor_event_data_t* data = (sensor_event_data_t*) event_data;
                   ESP_LOGI(TAG, "Data: Temp=%.1f C", data->temperature);
                   break;
               }
           }
       }
   }

   void app_main(void) {
       // 1. Khởi tạo Default Event Loop của hệ thống
       esp_event_loop_create_default();

       // 2. Đăng ký nhận TẤT CẢ signal thuộc bus APP_SYS_EVENTS
       esp_event_handler_instance_register(
           APP_SYS_EVENTS, 
           ESP_EVENT_ANY_ID, 
           &sys_event_handler, 
           NULL, 
           NULL
       );
   }

---

.. _section-production-best-practices:

IV. Chuẩn Định Hướng Cho Sản Phẩm (Production Best Practices)
=============================================================

Để đảm bảo hệ thống đạt độ ổn định cao trên môi trường Production, nên áp dụng thêm 3 thiết kế nâng cao sau:

1. Dùng Custom Dedicated Event Loop
-----------------------------------
Mặc định ``esp_event_loop_create_default()`` sẽ dùng chung Task cho cả hệ thống Wi-Fi/Bluetooth của ESP-IDF. Với các ứng dụng lớn, hãy tạo một Event Loop riêng bằng ``esp_event_loop_create()`` để tùy chỉnh Task Priority và Stack Size độc lập cho Application level.

2. Áp Dụng Offloading Pattern Cho Subscriber
---------------------------------------------
Nếu Subscriber cần thực hiện tác vụ nặng (ghi NVS, gửi HTTP Request, xử lý file):
* In Event Handler: Chỉ đẩy dữ liệu/event ID vào một FreeRTOS Queue (``xQueueSend``).
* Tạo một Worker Task riêng biệt chờ nhận dữ liệu từ Queue đó để xử lý bất đồng bộ, giải phóng Event Loop ngay lập tức.

3. Tối Ưu CMake Component Dependencies
---------------------------------------
Trong file ``CMakeLists.txt`` của các Component con (như ``wifi_manager``, ``sensor``), chỉ khai báo phụ thuộc vào ``event_bus``:

.. code-block:: cmake

   # File: components/wifi_manager/CMakeLists.txt
   idf_component_register(
       SRCS "wifi_manager.c"
       INCLUDE_DIRS "include"
       PRIV_REQUIRES event_bus
   )

Việc này đảm bảo tính đóng gói tuyệt đối, giảm thiểu bộ nhớ biên dịch và thời gian build project.