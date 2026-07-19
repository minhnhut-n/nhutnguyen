.. _general-http-req-resp:

========================================
Tổng Quan Về HTTP Request & Response
========================================

.. rubric:: I. Tổng Quan Về Mô Hình Client - Server

Mạng Internet hoạt động dựa trên nguyên tắc **Yêu cầu - Phản hồi (Request - Response)** giữa hai thực thể:

- **Client (Phía khách)**: Là bên chủ động đưa ra yêu cầu (Trình duyệt Web, Ứng dụng điện thoại, phần mềm Postman).
- **Server (Phía máy chủ)**: Là bên luôn ở trạng thái "lắng nghe" và chờ đợi yêu cầu gửi đến để xử lý rồi trả kết quả về (Trong dự án của bạn, ESP32 chính là Server).

.. rubric:: II. HTTP Request (Yêu cầu từ Client)

Trước hết một Request hoàn chỉnh cần đi qua những phần nào / layer nào?

::

   [ Trình duyệt / Client ]
          │  (1) Gửi HTTP Request (VD: POST /api/led-control với body là JSON)
          ▼
   [ Sóng Wi-Fi / Antenna ESP32 ]
          │  (2) Nhận sóng vô tuyến (Lớp Vật lý / Liên kết dữ liệu)
          ▼
   [ LwIP Stack (TCP/IP tích hợp sẵn trong ESP-IDF) ]
          │  (3) Bóc tách gói tin TCP, ghép các mảnh dữ liệu lại thành chuỗi thô (Raw String)
          ▼
   [ Component `esp_http_server` ]
          │  (4) Nhận chuỗi thô, phân tích (Parse) xem:
          │      - Method là gì? (GET, POST)
          │      - URI là gì? (/api/...)
          │      - Tìm Handler tương ứng đã đăng ký.
          ▼
   [ Hàm C Handler của bạn ] (User Application)
             (5) Bạn đọc dữ liệu thô, dùng thư viện (như cJSON) để biến đổi thành dữ liệu kiểu C (int, bool).

Phần dữ liệu được gửi xuống ESP32, có thể tồn tại nhiều cấu trúc dữ liệu khác nhau, vì vậy ``application/json`` được coi là một label để cho biết dạng dữ liệu là gì.

Đây là bảng các label có thể dùng để phân loại các thông tin khi nói đến HTML.

.. rubric:: Danh sách các Content-Type (MIME Types) thông dụng

.. list-table:: Bảng tổng hợp MIME Types trên ESP-IDF
   :widths: 15 35 50
   :header-rows: 1

   * - Nhóm
     - Cái nhãn (``Content-Type``)
     - Bản chất / Dùng làm gì trên ESP-IDF
   * - **DỮ LIỆU**
     - ``application/json``
     - Truyền nhận dữ liệu cấu trúc (tọa độ Joystick, mảng dữ liệu).
   * -
     - ``text/plain``
     - Chuỗi chữ thô, siêu nhẹ (gửi trạng thái ``ON``/``OFF``, số nhiệt độ).
   * -
     - ``application/octet-stream``
     - Dữ liệu nhị phân thô (Dùng khi nạp file Firmware ``.bin`` qua OTA).
   * - **GIAO DIỆN**
     - ``text/html``
     - Mã nguồn bộ khung giao diện Web (trả về tại đường dẫn gốc ``/``).
   * -
     - ``text/css``
     - File định dạng màu sắc, bố cục, giao diện nút bấm.
   * -
     - ``application/javascript``
     - File chứa code xử lý logic, gửi nhận dữ liệu ngầm ở Client.
   * - **HÌNH ẢNH**
     - ``image/png``
     - Trả về file ảnh định dạng PNG (logo, icon hiển thị trên Web).
   * -
     - ``image/jpeg``
     - Trả về file ảnh định dạng JPG/JPEG.
   * -
     - ``image/x-icon``
     - File icon nhỏ hiển thị trên tab của trình duyệt (``favicon.ico``).
   * - **FORM**
     - ``application/x-www-form-urlencoded``
     - Dữ liệu Form dạng nối chuỗi ``key=value`` (Cấu hình SSID/PASS Wi-Fi).
   * -
     - ``multipart/form-data``
     - Form chia nhiều phần (Dùng khi upload file cấu hình lên bộ nhớ Flash).

.. note::
   Trong ESP-IDF, bạn thiết lập nhãn này bằng hàm:
   ``httpd_resp_set_type(req, "tên_nhãn");``

.. rubric:: Ví dụ về cấu trúc dữ liệu JSON (application/json)

Khi truyền nhận dữ liệu qua API (ví dụ: đọc tọa độ Joystick), chuỗi dữ liệu dạng văn bản (String) truyền đi sẽ có cấu trúc như sau:

.. code-block:: json

   {
     "joy_x": 2048,
     "joy_y": 1012,
     "is_active": true
   }

.. note::
   * Các từ khóa (Key) như ``"joy_x"`` bắt buộc phải đặt trong dấu nháy kép ``"..."``.
   * Không được có dấu phẩy ``,`` ở phần tử cuối cùng (sau chữ ``true``).

.. rubric:: III. Quy trình gửi và nhận

Trong quá trình gửi (Serialization) từ bên front-end (web) và nhận (Deserialization) phía back-end, dữ liệu trong quá trình truyền phải là dạng string (dạng văn bản) nên cả hai bên phải tuân thủ quy tắc đóng gói và giải gói.

Phía gửi, dù có format đẹp như thế nào thì định dạng gửi đi luôn là oneline:

.. code-block:: json

   {"x":2048,"y":1012,"btn":1}

Phía nhận được cấu hình coi là json, javascript trên web sẽ parse ra thông tin của message đó:

.. code-block:: javascript

   let data = JSON.parse(raw_response);
   drawJoystickPointer(data.x, data.y);

.. rubric:: Luồng xử lý dữ liệu JSON (Giao thức truyền thông)

::

   Trình duyệt (JS)             ESP32 (ESP-IDF)
        |                             |
        |  ---[ HTTP POST / JSON ]--> | 1. Nhận chuỗi thô vào rx_buffer
        |                             | 2. cJSON_Parse() phân tách chuỗi
        |                             | 3. Trích xuất giá trị cấu hình
        |                             | 4. cJSON_Delete() giải phóng RAM
        |                             |
        |  <--[ HTTP RESP / JSON ]--- | 1. cJSON_CreateObject() dựng cấu trúc
        |                             | 2. cJSON_PrintUnformatted() nén chuỗi
        |                             | 3. httpd_resp_send() đẩy dữ liệu thô
        | 1. Nhận chuỗi chữ           |
        | 2. JSON.parse() bóc tách    |
        | 3. Cập nhật giao diện       |

.. rubric:: Quy tắc xử lý bộ nhớ trên ESP32

.. warning::
   Do thư viện ``cJSON`` sử dụng cơ chế cấp phát bộ nhớ động (Heap), sau khi kết thúc quá trình phân tách (Parse) dữ liệu nhận được, bắt buộc
   phải gọi hàm ``cJSON_Delete(root_object)`` để giải phóng bộ nhớ, tránh gây crash chip do tràn RAM.