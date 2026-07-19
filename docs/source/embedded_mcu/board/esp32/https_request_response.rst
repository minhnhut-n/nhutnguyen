2. HTTP Structure and Communication
===================================

.. rubric:: Tổng quan

- Tổng quan về quy trình giao tiếp giữa ESP32, webserver và backend.
- Xây dựng hàm endpoint HTTP và đăng ký handler trên ESP32.
- Cấu trúc webserver trên ESP32.
- Quy trình giao tiếp protocol giữa client và server.
- Ví dụ webserver cơ bản và HTTP request trong chế độ đơn giản.
- Triển khai đầy đủ và hoàn chỉnh.

.. rubric:: 1. Tổng quan về quy trình giao tiếp ESP32 - Webserver - Backend

Hệ thống điều khiển qua Web Dashboard trên ESP32 bao gồm 2 thực thể chính hoạt động thông qua kiến trúc Client-Server:

*Phía Web (Client Browser):*
    - **Front-end:** Gồm mã nguồn HTML (xây dựng cấu trúc layout hiển thị, các form, hộp thoại chứa dữ liệu) và CSS (định dạng màu sắc, bo góc card, nút bấm thành công/cảnh báo) để dựng giao diện GUI.
    - **Back-end ứng dụng (Javascript):** Đóng vai trò trung gian quản lý và vận hành GUI. JS tách biệt hoàn toàn khỏi HTML, có nhiệm vụ bắt các sự kiện click, tự động lấy thông tin từ các ô nhập dữ liệu (Form boxes), đóng gói thành chuỗi JSON và thực hiện các kỹ thuật bất đồng bộ (Fetch API/AJAX) để gửi/nhận dữ liệu ngầm với ESP32 mà không làm tải lại trang.

*Phía Back-end (ESP32 MCU):*
    - Vừa thực thi các tác vụ phần cứng cốt lõi của vi điều khiển (đọc cảm biến ADC, điều khiển rơ-le GPIO, xử lý logic hệ thống).
    - Vừa chạy một bộ định tuyến HTTP Daemon để lắng nghe các tín hiệu mạng, bóc tách chuỗi dữ liệu JSON từ Client gửi lên để điều khiển thiết bị, đồng thời gom trạng thái hệ thống thành định dạng JSON để trả ngược về giao diện.

.. code-block:: text

   +-------------------------------------------------------------------------------+
   |                           TRÌNH DUYỆT WEB (CLIENT)                            |
   |                                                                               |
   |   +-----------------------+              +--------------------------------+   |
   |   |   FRONT-END GUI       |              |   BACK-END ỨNG DỤNG (JS)       |   |
   |   |                       |              |                                |   |
   |   |   [HTML Khung Layout] |              |   - Bắt sự kiện Click / Form   |   |
   |   |   [CSS Giao diện]     |              |   - Đóng gói dữ liệu JSON      |   |
   |   |                       |              |   - Fetch API (Bất đồng bộ)    |   |
   |   +-----------+-----------+              +---------------+----------------+   |
   |               ^                                          |                    |
   |               | (Cập nhật UI)                            |                    |
   |               +------------------------------------------+                    |
   +----------------------------------------------------------|--------------------+
                                                              |
                                                    Giao thức HTTP (Wi-Fi)
                                             [GET / POST với application/json]
                                                              |
                                                              v
   +-------------------------------------------------------------------------------+
   |                            ESP32 MCU (SERVER BACK-END)                        |
   |                                                                               |
   |   +-----------------------------------------------------------------------+   |
   |   |   HTTP DAEMON ROUTER (Mạng & Định tuyến)                              |   |
   |   |                                                                       |   |
   |   |   - Kiểm tra Endpoint matching ( /, /script.js, /api/<name> )         |   |
   |   |   - Cơ chế LRU Purge dọn dẹp Socket cứu RAM khi đầy tải               |   |
   |   +-----------------------------------+-----------------------------------+   |
   |                                       |                                       |
   |                                       | (Kích hoạt Callback Handler)          |
   |                                       v                                       |
   |   +-----------------------------------------------------------------------+   |
   |   |   C CORE LOGIC & PHẦN CỨNG (Xử lý nhúng)                              |   |
   |   |                                                                       |   |
   |   |   - Sử dụng cJSON để giải mã lệnh nhận hoặc đóng gói trạng thái       |   |
   |   |   - Điều khiển Ngoại vi (GPIO Rơ-le, Đọc cảm biến ADC, Reboot)        |   |
   |   +-----------------------------------------------------------------------+   |
   +-------------------------------------------------------------------------------+

.. rubric:: 2. Xây dựng hàm endpoint HTTP và đăng ký handler

*   **Khái niệm Endpoint & Handler:** Một Endpoint (Điểm cuối) là một đường dẫn định tuyến dạng một chuỗi định danh (Ví dụ: ``/api/ping``, ``/api/relay``). Một Handler là một hàm Callback chạy trên ngôn ngữ C thuộc ESP-IDF, được gán cố định với Endpoint đó kèm theo phương thức HTTP xác định (GET hoặc POST).
*   **Định nghĩa hàm xử lý:** Các hàm callback bắt buộc phải tuân theo nguyên mẫu trả về kiểu ``esp_err_t`` và nhận vào một con trỏ quản lý ngữ cảnh kết nối cấu trúc ``httpd_req_t *req``.
*   **Cơ chế đăng ký:** Việc liên kết Endpoint và hàm Callback được thực hiện qua cấu trúc cấu hình ``httpd_uri_t``. ESP-IDF hỗ trợ đăng ký động các cấu trúc này tại thời điểm Runtime ngay sau khi máy chủ mạng khởi động thành công.

.. code-block::c
    // End point, GET method
    esp_err_t dashboard_get_handler(httpd_req_t *req) {
        httpd_resp_set_type(req, GUI_TYPE_HTML);
        httpd_resp_set_hdr(req, "Access-Control-Allow-Origin", "*");
        httpd_resp_sendstr(req, dashboard_html);
        return ESP_OK;
    }

    // End point, GET: /api/data 
    esp_err_t json_get_data(httpd_req_t *req) {
        cJSON *root = cJSON_CreateObject();
        cJSON_AddNumberToObject(root, "sensor", 25);
        cJSON_AddStringToObject(root, "state", "ok");
        esp_err_t ret = json_response_https(req, root);
        cJSON_Delete(root);
        return ret;
    }

    //register URI handler at esp_server start
    if (httpd_start(&server, &config) == ESP_OK) {
        //registry URI handler, in runtime (possible)
        httpd_uri_t uri_s[] = {
            {.uri = ROOT_URI, .method = HTTP_GET, .handler = dashboard_get_handler},
            {.uri = FAVICON_URI, .method = HTTP_GET, .handler = favicon_get_handler},

        };
        //registry one by one
        for (int i=0; i< sizeof(uri_s)/sizeof(uri_s[0]); i++) {
            httpd_register_uri_handler(server, &uri_s[i]);
        }
    };


.. rubric:: 3. Cấu trúc webserver trên ESP32

Để vận hành một hệ thống ổn định trên dòng vi điều khiển có tài nguyên RAM hạn chế như ESP32, cấu trúc Webserver cần tuân thủ các quy tắc tổ chức nghiêm ngặt:

*   **Tách biệt Module Trách nhiệm:** Toàn bộ file giao diện vật lý như ``index.html`` và ``script.js`` cần được lưu trữ độc lập dưới dạng tệp văn bản thô trong thư mục mã nguồn. Hệ thống biên dịch CMake sẽ nhúng trực tiếp các tệp này vào phân vùng Flash thành các khối dữ liệu nhị phân (Binary Blobs) thông qua từ khóa ``EMBED_TXTFILES``. Điều này giúp giải phóng hoàn toàn bộ nhớ RAM tĩnh.

Cmake configuration (/src/CMakeLists.txt):

.. code-block::cmake

    SET (SOURCE ${CMAKE_SOURCE_DIR}/src/main.c
                    ${CMAKE_SOURCE_DIR}/lib/wifi_api.c
                    ${CMAKE_SOURCE_DIR}/lib/http_json_helper.c
                    ${CMAKE_SOURCE_DIR}/lib/dashboard_page.c)

    SET (INCLUDE_DIR ${CMAKE_SOURCE_DIR}/src
                    ${CMAKE_SOURCE_DIR}/lib/)

    idf_component_register(
        SRCS ${SOURCE}
        INCLUDE_DIRS ${INCLUDE_DIR}
        PRIV_REQUIRES nvs_flash esp_netif esp_wifi esp_event
    )


*   **Xử lý các Endpoint Bắt buộc:** Bất kể kiến trúc nào, Webserver phải đăng ký tối thiểu hai Endpoint cơ bản để tối ưu hiệu suất mạng:
    1. Đường dẫn gốc (``/``): Trả về nội dung bộ khung HTML của Dashboard.
    2. Đường dẫn biểu tượng (``/favicon.ico``): Đây là Endpoint mà hầu hết trình duyệt Client luôn tự động gửi ngầm một yêu cầu độc lập để tìm icon hiển thị trên tab. Nếu ESP32 không đăng ký và bắt sự kiện này, chip sẽ báo lỗi thiếu callback (Warning Matching URI), gây nghẽn luồng xử lý hoặc trả về mã lỗi 404 không đáng có.

.. rubric:: 4. Protocol communication process

Quy trình bắt tay trao đổi giao thức HTTP giữa Client và ESP32 tuân theo các chu kỳ khép kín:

1.  **Gửi Request:** Trình duyệt Client tạo một kết nối Socket đến IP của ESP32, gửi kèm thông tin Header (ví dụ chỉ định rõ ``Content-Type: application/json``) và Payload (nếu là phương thức POST gửi lệnh cấu hình).

2.  **Phương thức sử dụng:**
    - ``GET``: Sử dụng khi Client muốn kéo dữ liệu trạng thái từ ESP32 về hiển thị lên màn hình.
    - ``POST``: Sử dụng khi Client đẩy lệnh thực thi, thay đổi trạng thái chân IO hoặc yêu cầu Reboot.

3.  **Xử lý phản hồi (Response):** ESP32 bóc tách dữ liệu thông qua các thư viện parse JSON (như cJSON), chuyển đổi trạng thái phần cứng tương ứng, sau đó thiết lập mã trạng thái phản hồi HTTP thích hợp (như ``200 OK`` cho dữ liệu thành công, hoặc ``204 No Content`` cho phản hồi Favicon nhanh chóng để đóng luồng mạng tức thì).

.. rubric:: 5. Sample basic webserver and HTTP request (general all process - basic mode)

Dưới đây là mã nguồn khởi tạo Webserver cơ bản, minh họa toàn bộ quy trình cấu hình, kích hoạt tính năng dọn dẹp bộ nhớ LRU, thiết lập Endpoint gốc ``/`` để phục vụ tệp HTML nhúng, và Endpoint xử lý Favicon tự động.

.. code-block:: c

    // Create end point handler
    // End point, GET: /api/data 
    esp_err_t json_get_data(httpd_req_t *req) {
        cJSON *root = cJSON_CreateObject();
        cJSON_AddNumberToObject(root, "sensor", 25);
        cJSON_AddStringToObject(root, "state", "ok");
        esp_err_t ret = json_response_https(req, root);
        cJSON_Delete(root);
        return ret;
    }

    // Init http and register URI handler (endpoint) at runtime
    esp_err_t start_http_server(void) {
        httpd_handle_t server = NULL;
        httpd_config_t config = HTTPD_DEFAULT_CONFIG();
        config.lru_purge_enable = true;
        config.max_uri_handlers = 16;
        
        if (httpd_start(&server, &config) == ESP_OK) {
            //registry URI handler, in runtime (possible)
            httpd_uri_t uri_s[] = {
                {.uri = ROOT_URI, .method = HTTP_GET, .handler = dashboard_get_handler},
            };

            //registry one by one
            for (int i=0; i< sizeof(uri_s)/sizeof(uri_s[0]); i++) {
                httpd_register_uri_handler(server, &uri_s[i]);
            }
        }
        return ESP_OK;
    }

.. rubric:: 6. Full implementation

Vể cơ bản, nếu đã thực hiện các bước trên thì chỉ cần lấy IP của esp (do router cung cấp). Ta có thể viết một test bench gửi request xuống
ESP thông qua json format đã cấu hình đó. Việc có host trên ESP hay không chỉ là hình thức mượn tạm Resource để chạy server.

Ví dụ code dashboard HTML có nhúng javascript để gọi các endpoint REST API của ESP32, hiển thị trạng thái và điều khiển rơ-le, trong code c
như sau. Lưu ý rằng trong thực tế, bạn nên tách riêng file HTML và JS ra để dễ quản lý, nhưng ở đây để minh họa, chúng ta nhúng trực tiếp vào mã nguồn C.
    
.. code-block:: c

    static const char *dashboard_html =
        "<!DOCTYPE html>"
        "<html lang=\"en\">"
        "<head>"
        "  <meta charset=\"UTF-8\" />"
        "  <meta name=\"viewport\" content=\"width=device-width, initial-scale=1.0\" />"
        "  <title>ESP-IDF Dashboard</title>"
        "  <style>"
        "    body{font-family:Arial,sans-serif;background:#0f172a;color:#e2e8f0;margin:0;padding:24px;}"
        "    .card{max-width:720px;margin:0 auto;background:#111827;border:1px solid #334155;border-radius:16px;padding:24px;}"
        "    h1{margin:0 0 8px;}"
        "    .muted{color:#94a3b8;}"
        "    button{margin:6px 6px 0 0;padding:10px 14px;border:none;border-radius:10px;background:#38bdf8;color:white;cursor:pointer;}"
        "    button.secondary{background:#1f2937;}"
        "    button.success{background:#22c55e;}"
        "    button.danger{background:#ef4444;}"
        "    pre{background:#020617;padding:14px;border-radius:12px;white-space:pre-wrap;word-break:break-word;}"
        "  </style>"
        "</head>"
        "<body>"
        "  <div class=\"card\">"
        "    <h1>ESP-IDF Dashboard</h1>"
        "    <p class=\"muted\">Basic internal dashboard for the current firmware REST API.</p>"
        "    <button onclick=\"call('/api/ping')\">Ping</button>"
        "    <button onclick=\"call('/api/data')\" class=\"secondary\">Get Data</button>"
        "    <button onclick=\"call('/api/relay', 'POST', '{\"relay\":1,\"state\":true}')\" class=\"success\">Relay ON</button>"
        "    <button onclick=\"call('/api/relay', 'POST', '{\"relay\":1,\"state\":false}')\" class=\"danger\">Relay OFF</button>"
        "    <button onclick=\"call('/api/reboot', 'POST')\" class=\"danger\">Reboot</button>"
        "    <pre id=\"out\">Click a button to test the ESP REST API.</pre>"
        "    <script>"
        "      function call(path, method='GET', body='') {"
        "        const ip = prompt('ESP IP address', '192.168.1.40');"
        "        if (!ip) return;"
        "        const url = 'http://' + ip + path;"
        "        const options = {method: method, headers: {'Content-Type':'application/json'}};"
        "        if (body) options.body = body;"
        "        fetch(url, options).then(async (res) => {"
        "          const text = await res.text();"
        "          document.getElementById('out').textContent = 'Status: ' + res.status + '\\n\\n' + text;"
        "        }).catch((err) => {"
        "          document.getElementById('out').textContent = 'Request failed: ' + err.message;"
        "        });"
        "      }"
        "    </script>"
        "  </div>"
        "</body>"
        "</html>";