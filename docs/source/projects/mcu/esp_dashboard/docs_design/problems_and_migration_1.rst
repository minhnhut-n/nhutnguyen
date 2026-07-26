.. _current-problems-and-migration-plan:

Problem And Migration Plan 1
==================================

.. rst-class:: lead

Tài liệu này phân tích **các vấn đề hiện tại** trong kiến trúc firmware ESP32 (ESP-IDF + C)
và đưa ra **lộ trình chuyển đổi** sang một kiến trúc Event-Driven dựa trên Event Bus và FreeRTOS.

.. rubric:: 1. Bối cảnh

Trong quá trình phát triển các sản phẩm sử dụng ESP32 với ESP-IDF, phần lớn mã nguồn được viết bằng C.
Điều này mang lại hiệu năng cao và dễ dàng tích hợp với SDK của Espressif, tuy nhiên cũng dẫn đến một số
vấn đề khi quy mô dự án ngày càng lớn.

Các thư viện (module/package) ban đầu thường được thiết kế với mục tiêu:

* **Portable** — có thể tái sử dụng ở nhiều dự án
* **Reusable** — ít phụ thuộc vào môi trường bên ngoài
* **Dễ bảo trì và kiểm thử**

Tuy nhiên, trong quá trình phát triển, việc bổ sung tính năng mới thường dẫn đến việc các module bắt đầu
gọi trực tiếp function của nhau, hoặc include chéo header của nhau để sử dụng một vài API.

.. code-block:: text
   :caption: Ví dụ — dependency hiện tại (trước khi chuyển đổi)

   HTTP Server
       │
       ├── gọi Storage
       ├── gọi WiFi Manager
       └── gọi WebSocket

   WiFi Manager
       └── gọi Storage

   Storage
       └── gọi WiFi Manager

Theo thời gian, các dependency này ngày càng nhiều và tạo thành mạng lưới phụ thuộc phức tạp.

--------------------------------------------------------------------------------

.. rubric:: 2. Vấn đề

Kiến trúc trên dẫn đến nhiều hệ quả:

.. rubric:: 2.1 Coupling cao

Các module biết quá nhiều về nhau.

.. code-block:: text

   http_server.c
       ↓
   storage.c
       ↓
   wifi_manager.c
       ↓
   websocket.c

Nếu thay đổi Storage, rất có thể WiFi hoặc HTTP cũng cần sửa theo.

.. warning::

   **Coupling cao** làm tăng thời gian bảo trì và nguy cơ giới thiệu lỗi mới
   khi chỉnh sửa một module duy nhất.

.. rubric:: 2.2 Mất tính Portable

Một thư viện vốn chỉ có nhiệm vụ quản lý WiFi nhưng lại phải include:

.. code-block:: c

   #include "http_server.h"
   #include "websocket.h"
   #include "storage.h"

Thư viện này gần như không còn khả năng tái sử dụng ở project khác.

.. rubric:: 2.3 Khó mở rộng

Mỗi lần thêm một chức năng mới thường phải:

* include thêm header
* gọi thêm function
* sửa nhiều module cùng lút

Thay vì mở rộng độc lập, các module ngày càng phụ thuộc vào nhau.

.. rubric:: 2.4 Khó Debug

Khi phát sinh lỗi:

.. code-block:: text

   HTTP
       ↓
   Storage
       ↓
   WiFi
       ↓
   Network

Rất khó xác định lỗi xuất phát từ đâu. Một lỗi có thể bị gây ra bởi rất nhiều module khác nhau.

.. rubric:: 2.5 Khó kiểm thử (Unit Test)

Do các module phụ thuộc trực tiếp vào nhau nên rất khó viết test độc lập.

.. code-block:: text

   Muốn test Storage nhưng Storage lại phụ thuộc WiFi.
   Muốn test WiFi nhưng WiFi lại gọi HTTP.

.. admonition:: Tổng kết các vấn đề
   :class: danger

   | **Coupling cao** → khó bảo trì
   | **Mất Portable** → khó tái sử dụng
   | **Khó mở rộng** → chậm tiến độ
   | **Khó Debug** → tốn thời gian sửa lỗi
   | **Khó Test** → giảm chất lượng

--------------------------------------------------------------------------------

.. rubric:: 3. Mục tiêu

Mục tiêu là xây dựng kiến trúc mà trong đó:

* Các module **không gọi trực tiếp** nhau
* Mỗi module chỉ tập trung xử lý **đúng trách nhiệm** của mình
* Có thể **thêm hoặc bỏ module** mà không ảnh hưởng các module còn lại
* Tăng **khả năng tái sử dụng** và **bảo trì**

--------------------------------------------------------------------------------

.. rubric:: 4. Hướng giải quyết

Thay vì module A gọi trực tiếp module B, module A chỉ phát ra một **"tín hiệu" (Signal/Event)**.
Module nào quan tâm sẽ tự đăng ký nhận tín hiệu đó và xử lý.

.. code-block:: text
   :caption: Kiến trúc Event Bus (sau khi chuyển đổi)

   HTTP POST
         │
         ▼
   Publish Event
         │
         ▼
   +------------------+
   |    Event Bus     |
   +------------------+
       │
       ├────────────► Storage
       │
       ├────────────► WiFi
       │
       └────────────► WebSocket

HTTP không biết Storage. HTTP không biết WiFi. HTTP chỉ biết "đã có cấu hình WiFi mới".

--------------------------------------------------------------------------------

.. rubric:: 5. Event Bus

Để hiện thực mô hình trên, xây dựng một thư viện trung gian gọi là **Event Bus**.

Vai trò của Event Bus:

* **Publish Event** (phát sự kiện)
* **Subscribe Event** (đăng ký nhận sự kiện)
* **Dispatch Event** (chuyển sự kiện tới đúng module)

Mỗi package chỉ cần giao tiếp với Event Bus thay vì giao tiếp trực tiếp với nhau.

.. code-block:: text

   HTTP
      │
   Publish
      │
   Event Bus
      │
   ┌─┴──────────────┐
   │                │
   Storage       WiFi Manager

--------------------------------------------------------------------------------

.. rubric:: 6. Mỗi module hoạt động độc lập

Ví dụ:

.. code-block:: text

   HTTP nhận được:

   {
       "ssid":"Home",
       "password":"12345678"
   }

HTTP chỉ làm:

.. code-block:: text

   Publish WIFI_CONFIG_RECEIVED

Storage:

.. code-block:: text

   Subscribe WIFI_CONFIG_RECEIVED
   ↓
   Save NVS

WiFi Manager:

.. code-block:: text

   Subscribe WIFI_CONFIG_RECEIVED
   ↓
   Reconnect WiFi

WebSocket:

.. code-block:: text

   Subscribe WIFI_CONFIG_RECEIVED
   ↓
   Notify Client

HTTP hoàn toàn không cần biết các module khác tồn tại.

--------------------------------------------------------------------------------

.. rubric:: 7. Vai trò của FreeRTOS

Một vấn đề khác là nhiều project C truyền thống không sử dụng FreeRTOS đúng cách.

Luồng xử lý thường có dạng:

.. code-block:: c

   while(1)
   {
       http_server();
       wifi_manager();
       websocket();
       storage();
       uart();
   }

Đây là mô hình xử lý tuần tự.

.. list-table:: Nhược điểm của mô hình tuần tự
   :header-rows: 1
   :widths: 50 50

   * - Nhược điểm
     - Mô tả
   * - Blocking
     - Task này chờ task kia
   * - Không tận dụng được khả năng đa nhiệm
     - Của ESP32
   * - Dễ phát sinh deadlock hoặc starvation
     - Nếu thiết kế không tốt

--------------------------------------------------------------------------------

.. rubric:: 8. FreeRTOS làm nền tảng

ESP-IDF đã tích hợp sẵn FreeRTOS. Do đó nên xem mỗi thành phần là một **Task độc lập**.

.. code-block:: text
   :caption: Các Task trong hệ thống

   HTTP Task
   Storage Task
   WiFi Task
   UART Task
   WebSocket Task
   FFT Task

Mỗi Task chỉ xử lý công việc của mình. Không phụ thuộc trực tiếp vào các Task khác.

--------------------------------------------------------------------------------

.. rubric:: 9. Kết hợp Event Bus và FreeRTOS

Kiến trúc đề xuất (Event Bus + FreeRTOS):

.. code-block:: text

                     Event Bus
                          │
      ┌───────────────────┼────────────────────┐
      │                   │                    │
  HTTP Task          Storage Task         WiFi Task
      │                   │                    │
      └───────────────────┼────────────────────┘
                          │
                  WebSocket Task

Mỗi Task:

* **Subscribe** các Event mình quan tâm
* **Publish Event** khi hoàn thành công việc

--------------------------------------------------------------------------------

.. rubric:: 10. Trao đổi dữ liệu bằng Queue

Không phải Event nào cũng chỉ mang ý nghĩa thông báo. Nhiều trường hợp cần truyền dữ liệu.

.. code-block:: text
   :caption: Ví dụ truyền dữ liệu

   UART
       ↓
   Frame
       ↓
   FFT

   hoặc

   HTTP
       ↓
   SSID + Password
       ↓
   Storage

Trong trường hợp này có thể sử dụng:

* FreeRTOS Queue (``xQueue``)
* Message Queue
* Ring Buffer

.. tip::

   **Event** dùng để báo "đã có việc". **Queue** dùng để mang dữ liệu.

.. code-block:: text
   :caption: Luồng xử lý kết hợp Event và Queue

   HTTP
       ↓
   Publish WIFI_CONFIG_RECEIVED
       ↓
   Push Data vào Queue
       ↓
   Storage Task nhận Queue
       ↓
   Save NVS

--------------------------------------------------------------------------------

.. rubric:: 11. Kiến trúc đề xuất

Kiến trúc tổng thể đề xuất:

.. code-block:: text

                 +----------------+
                 |   Event Bus    |
                 +----------------+
                  ▲     ▲      ▲
                  │     │      │
       Publish ───┘     │      └──── Subscribe
                        │
        +---------------+----------------+
        |                                |
 +--------------+                +--------------+
 | HTTP Task    |                | WiFi Task    |
 +--------------+                +--------------+

 +--------------+                +--------------+
 | Storage Task |                | UART Task    |
 +--------------+                +--------------+

 +--------------+                +--------------+
 | WebSocket    |                | FFT Task     |
 +--------------+                +--------------+

           ▲
           │
        xQueue

--------------------------------------------------------------------------------

.. rubric:: 12. Lợi ích

**Tính độc lập**

Mỗi module chỉ quan tâm tới nhiệm vụ của chính nó.

**Dễ mở rộng**

Thêm module mới chỉ cần:

* Subscribe Event
* Không cần sửa các module cũ

**Dễ bảo trì**

Các dependency giữa các module giảm đáng kể.

**Dễ Debug**

Có thể theo dõi luồng Event để biết dữ liệu đi qua đâu.

**Dễ kiểm thử**

Có thể mock Event Bus hoặc Queue để test từng module độc lập.

**Tái sử dụng**

Một module không còn phụ thuộc vào HTTP, WiFi hay WebSocket. Có thể mang nguyên module sang project khác.

--------------------------------------------------------------------------------

.. rubric:: 13. Kết luận

Đối với các dự án ESP32 sử dụng ESP-IDF, đặc biệt khi số lượng chức năng ngày càng tăng (HTTP Server,
WiFi, NVS, UART, FFT, WebSocket, OTA, MQTT...), việc các module gọi trực tiếp lẫn nhau sẽ nhanh chóng
làm hệ thống trở nên khó mở rộng và khó bảo trì.

Kiến trúc đề xuất là sử dụng FreeRTOS làm nền tảng thực thi, trong đó mỗi thành phần chạy dưới dạng
một Task độc lập. Việc giao tiếp giữa các thành phần được thực hiện thông qua Event Bus (để phát và
nhận sự kiện) kết hợp với FreeRTOS Queue (để truyền dữ liệu).

Cách tiếp cần này giúp giảm coupling giữa các module, tăng tính portable và reusable của thư viện, đồng
thời tạo nền tảng phù hợp để phát triển các sản phẩm có quy mô lớn và dễ mở rộng trong tương lai.

--------------------------------------------------------------------------------

.. rubric:: Lộ trình chuyển đổi

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
     Giai đoạn 3 – Tích hợp Event Bus
       : Xây dựng Event Bus
       : Chuyển direct call → Publish/Subscribe
       : Dùng Queue truyền dữ liệu
     Giai đoạn 4 – Nâng cao chất lượng
       : Viết Unit Test cho từng module
       : Tích hợp CI/CD
       : Code review kiến trúc

.. rubric:: Giai đoạn 1 – Chuẩn hóa cấu trúc

* Tách thư mục theo **Layer** (Application, Service, Middleware, Driver, HAL)
* Loại bỏ **include chéo** giữa các module
* Chuẩn hóa **naming convention** (file, event, queue, task)
* Thống nhất **lifecycle** của module (``init``, ``start``, ``stop``, ``reset``, ``deinit``)

.. rubric:: Giai đoạn 2 – Đưa FreeRTOS vào kiến trúc

* Chuyển các thành phần lớn (WiFi, HTTP, UART...) thành các **Task độc lập**
* Xây dựng **Service Manager** để quản lý vòng đời
* Áp dụng **State Machine** cho các service có nhiều trạng thái (WiFi, OTA, MQTT...)

.. rubric:: Giai đoạn 3 – Tích hợp Event Bus

* Xây dựng **Event Bus** với cơ chế Publish / Subscribe / Dispatch
* Chuyển các **direct call** thành **Publish Event**
* Dùng **FreeRTOS Queue** truyền dữ liệu lớn
* Đảm bảo **Event** chỉ mang ý nghĩa thông báo, **Queue** mới mang dữ liệu

.. rubric:: Giai đoạn 4 – Nâng cao chất lượng

* Viết **Unit Test** cho từng module (mock Event Bus và Queue)
* Tích hợp **CI/CD** tự động
* **Code review** kiến trúc định kỳ

.. note::

   Mỗi giai đoạn đều có thể triển khai độc lập và **không ảnh hưởng** tới các module đã hoàn thành
   ở các giai đoạn trước.

--------------------------------------------------------------------------------

.. rubric:: Kết luận

.. rst-class:: conclusion

   Chuyển đổi từ kiến trúc **"function call trực tiếp"** sang **"Event-Driven + FreeRTOS"**
   là một quá trình **dần dần và có hệ thống**. Với lộ trình trên, dự án có thể giảm dần coupling,
   tăng khả năng mở rộng và bảo trì mà **không cần ngừng phát triển** để đạt được kiến trúc hoàn hảo.

.. raw:: html

   <hr style="border: 1px solid #2980b9; margin: 2em 0;">
   <p style="text-align: center; color: #7f8c8d;">
   <em>Current Problems & Migration Plan v1.0</em>
   </p>
