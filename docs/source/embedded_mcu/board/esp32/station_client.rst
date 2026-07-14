Station & Client
=================

Hướng dẫn kết nối ESP32 ở chế độ **Station (STA)** với Wi-Fi và giao tiếp với client qua TCP.

.. rubric:: Tổng quan

- **ESP32 Station**: ESP32 kết nối vào mạng Wi-Fi có sẵn (Access Point).
- **TCP Server trên ESP32**: ESP32 chạy server socket, client kết nối tới để trao đổi dữ liệu.
- **Client**: Thiết bị (PC, điện thoại, MCU khác) kết nối tới ESP32 qua TCP.

.. image:: https://docs.espressif.com/projects/esp-idf/en/latest/esp32/_images/wifi-station.png
   :alt: ESP32 Station Mode
   :width: 400px

---

.. rubric:: 1. Cấu hình Wi-Fi Station

.. code-block:: c

   #include "esp_wifi.h"
   #include "esp_event.h"
   #include "nvs_flash.h"

   void wifi_init_sta(void)
   {
       // Khởi tạo NVS
       esp_err_t ret = nvs_flash_init();
       if (ret == ESP_ERR_NVS_NO_FREE_PAGES || ret == ESP_ERR_NVS_NEW_VERSION_FOUND) {
           ESP_ERROR_CHECK(nvs_flash_erase());
           ret = nvs_flash_init();
       }
       ESP_ERROR_CHECK(ret);

       // Khởi tạo network interface
       ESP_ERROR_CHECK(esp_netif_init());
       ESP_ERROR_CHECK(esp_event_loop_create_default());
       esp_netif_create_default_wifi_sta();

       // Khởi tạo Wi-Fi
       wifi_init_config_t cfg = WIFI_INIT_CONFIG_DEFAULT();
       ESP_ERROR_CHECK(esp_wifi_init(&cfg));

       // Cấu hình Station
       wifi_config_t wifi_config = {
           .sta = {
               .ssid = "YOUR_SSID",
               .password = "YOUR_PASSWORD",
               .threshold.authmode = WIFI_AUTH_WPA2_PSK,
           },
       };
       ESP_ERROR_CHECK(esp_wifi_set_mode(WIFI_MODE_STA));
       ESP_ERROR_CHECK(esp_wifi_set_config(WIFI_IF_STA, &wifi_config));
       ESP_ERROR_CHECK(esp_wifi_start());

       ESP_LOGI(TAG, "wifi_init_sta finished.");
   }

.. rubric:: 2. Xử lý sự kiện Wi-Fi

.. code-block:: c

   static void event_handler(void* arg, esp_event_base_t event_base,
                             int32_t event_id, void* event_data)
   {
       if (event_base == WIFI_EVENT && event_id == WIFI_EVENT_STA_START) {
           esp_wifi_connect();
       } else if (event_base == WIFI_EVENT && event_id == WIFI_EVENT_STA_DISCONNECTED) {
           esp_wifi_connect(); // Tự động kết nối lại
           ESP_LOGI(TAG, "Retry to connect to the AP");
       } else if (event_base == IP_EVENT && event_id == IP_EVENT_STA_GOT_IP) {
           ip_event_got_ip_t* event = (ip_event_got_ip_t*) event_data;
           ESP_LOGI(TAG, "Got IP: " IPSTR, IP2STR(&event->ip_info.ip));
       }
   }

Đăng ký event handler trong ``wifi_init_sta()``:

.. code-block:: c

   ESP_ERROR_CHECK(esp_event_handler_instance_register(
       WIFI_EVENT, ESP_EVENT_ANY_ID, &event_handler, NULL, NULL));
   ESP_ERROR_CHECK(esp_event_handler_instance_register(
       IP_EVENT, IP_EVENT_STA_GOT_IP, &event_handler, NULL, NULL));

---

.. rubric:: 3. TCP Server trên ESP32

Sau khi có IP, ESP32 có thể chạy một TCP server để client kết nối.

.. code-block:: c

   #include "lwip/err.h"
   #include "lwip/sockets.h"
   #include "lwip/sys.h"
   #include <lwip/netdb.h>

   #define TCP_PORT 3333

   void tcp_server_task(void *pvParameters)
   {
       char addr_str[128];
       int addr_family = AF_INET;
       int ip_protocol = IPPROTO_IP;

       struct sockaddr_in dest_addr;
       dest_addr.sin_addr.s_addr = htonl(INADDR_ANY);
       dest_addr.sin_family = AF_INET;
       dest_addr.sin_port = htons(TCP_PORT);

       int listen_sock = socket(addr_family, SOCK_STREAM, ip_protocol);
       if (listen_sock < 0) {
           ESP_LOGE(TAG, "Unable to create socket: errno %d", errno);
           vTaskDelete(NULL);
           return;
       }

       int err = bind(listen_sock, (struct sockaddr *)&dest_addr, sizeof(dest_addr));
       if (err != 0) {
           ESP_LOGE(TAG, "Socket unable to bind: errno %d", errno);
           goto CLEAN_UP;
       }

       err = listen(listen_sock, 1);
       if (err != 0) {
           ESP_LOGE(TAG, "Error occurred during listen: errno %d", errno);
           goto CLEAN_UP;
       }

       ESP_LOGI(TAG, "TCP server listening on port %d", TCP_PORT);

       while (1) {
           struct sockaddr_in source_addr;
           socklen_t addr_len = sizeof(source_addr);
           int sock = accept(listen_sock, (struct sockaddr *)&source_addr, &addr_len);
           if (sock < 0) {
               ESP_LOGE(TAG, "Unable to accept connection: errno %d", errno);
               break;
           }

           // Nhận dữ liệu từ client
           char rx_buffer[128];
           while (1) {
               int len = recv(sock, rx_buffer, sizeof(rx_buffer) - 1, 0);
               if (len < 0) {
                   ESP_LOGE(TAG, "recv failed: errno %d", errno);
                   break;
               } else if (len == 0) {
                   ESP_LOGI(TAG, "Connection closed");
                   break;
               } else {
                   rx_buffer[len] = 0;
                   ESP_LOGI(TAG, "Received %d bytes: %s", len, rx_buffer);

                   // Gửi phản hồi lại client
                   char response[] = "Hello from ESP32!\n";
                   send(sock, response, strlen(response), 0);
               }
           }

           close(sock);
       }

   CLEAN_UP:
       close(listen_sock);
       vTaskDelete(NULL);
   }

Khởi chạy task TCP server sau khi có IP:

.. code-block:: c

   // Trong event handler IP_EVENT_STA_GOT_IP
   xTaskCreate(tcp_server_task, "tcp_server", 4096, NULL, 5, NULL);

---

.. rubric:: 4. Client kết nối tới ESP32

**Python client (trên PC):**

.. code-block:: python

   import socket

   ESP32_IP = "192.168.1.100"  # Thay bằng IP của ESP32
   ESP32_PORT = 3333

   client = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
   client.connect((ESP32_IP, ESP32_PORT))

   # Gửi dữ liệu
   client.send(b"Hello ESP32!")

   # Nhận phản hồi
   response = client.recv(1024)
   print(f"ESP32 says: {response.decode()}")

   client.close()

**C client (trên Linux/MCU khác):**

.. code-block:: c

   #include <stdio.h>
   #include <string.h>
   #include <sys/socket.h>
   #include <arpa/inet.h>

   int main() {
       int sock = socket(AF_INET, SOCK_STREAM, 0);
       struct sockaddr_in server;

       server.sin_addr.s_addr = inet_addr("192.168.1.100");
       server.sin_family = AF_INET;
       server.sin_port = htons(3333);

       connect(sock, (struct sockaddr *)&server, sizeof(server));

       char *message = "Hello ESP32!";
       send(sock, message, strlen(message), 0);

       char response[128] = {0};
       recv(sock, response, sizeof(response), 0);
       printf("ESP32 says: %s\n", response);

       close(sock);
       return 0;
   }

---

.. rubric:: 5. Kiểm tra kết nối

.. code-block:: bash

   # Dùng netcat (nc) để test nhanh từ terminal
   nc 192.168.1.100 3333
   # Nhập "Hello" và Enter, ESP32 sẽ phản hồi

---

.. rubric:: 6. Lưu ý

- Đảm bảo ESP32 và client **cùng mạng LAN**.
- Có thể cần cấu hình **tĩnh IP** cho ESP32 để client dễ dàng kết nối.
- Sử dụng ``esp_netif_set_ip_info()`` để set IP tĩnh nếu cần.
- Với ứng dụng thực tế, nên thêm **xử lý timeout** và **reconnect** cho socket.

---

.. rubric:: Tài liệu tham khảo

- `ESP-IDF Wi-Fi Station Example <https://github.com/espressif/esp-idf/tree/master/examples/wifi/getting_started/station>`_
- `ESP-IDF TCP Server Example <https://github.com/espressif/esp-idf/tree/master/examples/protocols/sockets/tcp_server>`_
- `ESP32 Wi-Fi API Reference <https://docs.espressif.com/projects/esp-idf/en/latest/esp32/api-reference/network/esp_wifi.html>`_