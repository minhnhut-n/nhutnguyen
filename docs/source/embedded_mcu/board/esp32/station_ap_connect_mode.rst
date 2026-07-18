Station & Access Point Connect
==============================

Hướng dẫn cấu hình esp32 hoạt động trong hai chế độ STA và AP, ngoài ra còn hướng dẫn quá trình chuyển đổi
mode giữa hai chế độ.

.. rubric:: Tổng quan

- **ESP32 Station**: ESP32 kết nối vào mạng Wi-Fi có sẵn.
- **ESP32 Access Point**: ESP32 đóng vai trò là máy host, phát Wi-Fi cho các thiết bị (client) kết nối
- **Client**: Các thiết bị bất kì có thể làm máy khách kết nối vào Wi-Fi đang phát.

.. rubric:: Tổng quan về các bước cần thực hiện để một module hoạt động trên esp32

Để khởi tạo Wi-Fi Station, cần thực hiện các bước sau:

Bước 1: Khởi tạo non volatile memory
    Trong esp không có emmc hoặc eeprom để lưu các cấu hình như arduino nên trong hầu hết mọi kịch 
    bản sử dụng NVS là cần thiết để lưu trữ các thông tin config khi power shutdown cho lần tiếp threshold

Bước 2: Cấu hình network interface
    Cấu hình interface connect, việc cần làm hầu hết với các module connect (integrated trên board esp)
    
    Bước 2.1: Khởi tạo network interface, (esp_netif_init())

    Bước 2.2: Tạo event loop (do esp32 quản lý event theo asyn - bất đồng bộ mode), event loop giúp nhận
    diện được các event xảy ra trong quá trình thiết lập và sử dụng
    - (esp_event_loop_create_default())

    Các module ở tầng driver hoặc platform IDE sử dụng event loop để gửi tín hiệu cho system về các trạng 
    thái của bất kì một module nào (bluetooth, wifi, ...) nếu không có, thì có thể xảy ra SIGSEGV (Signal 6)
    --> BẮT BUỘC

    Bước 2.3: Khởi tạo network interface cho từng mode (specify, chọn 1 trong 2, với các module khác thì 
    tương tự)
    - (esp_netif_create_default_wifi_ap()) cho access point mode
    - (esp_netif_create_default_wifi_sta()) cho station mode

Bước 3: Cấu hình wifi stack
    Với tùy từng module, sẽ có từng stack cấu hình khác nhau, trong bài này cấu hình đó là wifi stack

    Bước 3.1: Khởi tạo wifi module với các config của nó (xem WIFI_INIT_CONFIG_DEFAULT()) để biết thêm về
    các config cần nạp vào module trước khi chạy (khởi tạo này là một lần duy nhất khi bật điện)

    Bước 3.2: Khởi tạo mode kết nối và nạp cấu hình thích hợp cho từng mode, trong ESP có 2 mode chính gồm:
    - AP mode: trong đó ESP đóng vai trò là máy phát, cho các client connect (nên chọn là default trong hầu 
    hết các IOT application)
    - STATION mode: trong đó ESP là một client, access đến các điểm phát khác
    Do đó sẽ có các thông số khác nhau cần cấu hình cho ESP.

    Bước 3.3: (Optional) Có thể đăng kí hàm event handler cho wifi tại thời điểm này.
    (esp_event_handler_instance_register())

Bước 4: Gửi config và mode được cấu hình vào module
    Gồm 02 step gồm:
    - Gửi thông tin mode được cấu hình (esp_wifi_set_mode)
    - Gửi thông tin config tổng hợp (esp_wifi_set_config)
    Thứ tự này cần đảm bảo thực hiện đúng thứ tự

Bước 5: Start module
    (esp_wifi_start())


.. rubric:: 1. Cấu hình Wi-Fi Access Point

.. code-block:: c

    const char* g_wifi_tag = "ESP_WIFI_C";
    esp_err_t wifi_init_general(void) {
        // for storing config
        esp_err_t ret = nvs_flash_init();
        if (ret == ESP_ERR_NVS_NO_FREE_PAGES || ret == ESP_ERR_NVS_NEW_VERSION_FOUND) {
            // raise another change
            nvs_flash_erase();
            ret = nvs_flash_init();
        }
        ESP_ERROR_CHECK(ret);

        //network interface
        ret = esp_netif_init();
        ret = esp_event_loop_create_default(); // system event (async)
        esp_netif_create_default_wifi_ap(); // dynamic allocate for ap mode config

        //wifi init
        wifi_init_config_t wifi_config = WIFI_INIT_CONFIG_DEFAULT();
        ret = esp_wifi_init(&wifi_config);

        //ap config
        g_wifi_mode = WIFI_MODE_AP;
        wifi_config_t config = {
            .ap = {
                .ssid = ESP_INTERNAL_SSID,
                .ssid_len = strlen(ESP_INTERNAL_SSID),
                .password = ESP_INTERNAL_PASS, 
                .max_connection = ESP_INTERNAL_STAM,
                .channel = ESP_INTERNAL_CHAN,
                .authmode = WIFI_AUTH_WPA2_PSK
            }
        };
        
        if (strlen(ESP_INTERNAL_PASS) == 0) {
            config.ap.authmode = WIFI_AUTH_OPEN;
        }

        //softAP and apply config
        ret = esp_wifi_set_mode(WIFI_MODE_AP);
        ret = esp_wifi_set_config(WIFI_IF_AP, &config);

        //start wifi
        ret = esp_wifi_start();
        ESP_LOGI(g_wifi_tag, "esp wifi start ap ready!");
        return ret;
    }

.. rubric:: 2. Xử lý sự kiện Wi-Fi và IP

.. code-block:: c

    //Tạo event handler
    static void esp_wifi_event_handler(void* arg, esp_event_base_t e_base, int32_t e_id, void* e_data) {
        if (e_base == WIFI_EVENT) {
            switch (e_id) {
                case WIFI_EVENT_SCAN_DONE: // Wi-Fi event declarations enum
                    ESP_LOGI(g_wifi_tag, "wifi SCAN done!");
                    break;

                case WIFI_EVENT_STA_START: // Wi-Fi event declarations enum
                    ESP_LOGI(g_wifi_tag, "wifi as station START!");
                    break;
                case WIFI_EVENT_STA_STOP: // Wi-Fi event declarations enum
                    ESP_LOGI(g_wifi_tag, "wifi as station STOP!");
                    break;
                case WIFI_EVENT_STA_CONNECTED: // Wi-Fi event declarations enum
                    ESP_LOGI(g_wifi_tag, "wifi as station CONNECTED!");
                    break;
                case WIFI_EVENT_STA_DISCONNECTED: // Wi-Fi event declarations enum
                    ESP_LOGI(g_wifi_tag, "wifi as station DISCONNECTED!");
                    break;

                case WIFI_EVENT_AP_START: // Wi-Fi event declarations enum
                    ESP_LOGI(g_wifi_tag, "wifi as AP START!");
                    break;
                case WIFI_EVENT_AP_STOP: // Wi-Fi event declarations enum
                    ESP_LOGI(g_wifi_tag, "wifi as AP STOP!");
                    break;
                case WIFI_EVENT_AP_STACONNECTED: // Wi-Fi event declarations enum
                    ESP_LOGI(g_wifi_tag, "wifi as AP CONNECTED!");
                    break;
                case WIFI_EVENT_AP_STADISCONNECTED: // Wi-Fi event declarations enum
                    ESP_LOGI(g_wifi_tag, "wifi as AP DISCONNECTED!");
                    break;

                default:
                    break;
            }
            return;
        }
        if (e_base == IP_EVENT) {
            switch (e_id) {
                case IP_EVENT_STA_GOT_IP: // IP event declarations enum
                    ESP_LOGI(g_wifi_tag, "wifi station got IP!");
                    break;
                case IP_EVENT_STA_LOST_IP: // IP event declarations enum
                    ESP_LOGI(g_wifi_tag, "wifi station lost IP!");
                    break;
                case IP_EVENT_TX_RX: // IP event declarations enum
                    ESP_LOGI(g_wifi_tag, "wifi on TRANSMISSION!");
                    break;
                default:
                    break;
            }
            return;
        }
    }

    /* bổ sung vào, wifi_init_general()*/
    //network interface
    ret = esp_netif_init();
    ret = esp_event_loop_create_default(); // system event (async)
    esp_netif_create_default_wifi_ap(); // dynamic allocate for ap mode config

    //esp_event_handler_t là một con trỏ hàm (*void)
    esp_event_handler_instance_register(WIFI_EVENT, ESP_EVENT_ANY_ID, &esp_wifi_event_handler, NULL, NULL);
    esp_event_handler_instance_register(IP_EVENT, ESP_EVENT_ANY_ID, &esp_wifi_event_handler, NULL, NULL);

    //wifi init
    wifi_init_config_t wifi_config = WIFI_INIT_CONFIG_DEFAULT();
    ret = esp_wifi_init(&wifi_config);

.. rubric:: 3. Cấu hình chuyển đổi sang STATION mode

Tương tự với cấu hình khởi tạo cho AP, một vài bước ta có thể loại bỏ không cần phải làm lại khi đã làm ở
header 1 rồi. Cách này có tính chất tương tự nên có thể đảo ngược lại vẫn bình thường, ví dụ (cấu hình ban đầu 
là STA sau đổi lại AP).

Main code:

.. code-block:: c

    #include "wifi_api.h"

    void app_main() {
        wifi_init_general();
        vTaskDelay(pdMS_TO_TICKS(3000));
        wifi_station_mode(NULL);
    }


.. code-block:: c

    #define TEMP_SSID "<fill_in_here>"
    #define TEMP_PASS "<fill_in_here>"
    esp_err_t wifi_station_mode(int* param) {
        esp_err_t ret;
        ret = esp_wifi_stop();

        g_wifi_mode = WIFI_MODE_STA;
        wifi_config_t config = {
            .sta = {
                .ssid = TEMP_SSID,
                .password = TEMP_PASS,
                .threshold.authmode = WIFI_AUTH_WPA2_PSK
            },
        };

        ret = esp_wifi_set_mode(WIFI_MODE_STA);
        ret = esp_wifi_set_config(WIFI_IF_STA, &config);

        ret = esp_wifi_start();
        ESP_LOGI(g_wifi_tag, "Switch mode to Wifi STA!");

        ret = esp_wifi_connect();
        ESP_LOGI(g_wifi_tag, "Connecting to AP...");
        return ret;
    }

Log

.. code-block:: bash

    I (701) ESP_WIFI_C: wifi as AP START!
    I (3701) wifi:flush txq
    I (3701) wifi:stop sw txq
    I (3701) wifi:lmac stop hw txq
    I (3701) ESP_WIFI_C: wifi as AP STOP!
    I (3711) wifi:mode : sta (mac code)
    I (3711) wifi:enable tsf
    I (3711) ESP_WIFI_C: wifi as station START!
    I (3711) ESP_WIFI_C: Switch mode to Wifi STA!
    I (3721) ESP_WIFI_C: Connecting to AP...
    I (3971) main_task: Returned from app_main()
    I (3981) wifi:new:<1,0>, old:<1,0>, ap:<255,255>, sta:<1,0>, prof:7, snd_ch_cfg:0x0
    I (3991) wifi:state: init -> auth (0xb0)
    I (3991) wifi:state: auth -> assoc (0x0)
    I (3991) wifi:state: assoc -> run (0x10)
    I (4001) wifi:connected with <fill_in_here>, aid = 3, channel 1, BW20, bssid = f4:0d:9f:cf:37:40
    I (4011) wifi:security: WPA2-PSK, phy: bgn, rssi: -52
    I (4031) wifi:pm start, type: 1

    I (4031) ESP_WIFI_C: wifi as station CONNECTED!
    I (4151) wifi:AP's beacon interval = 204800 us, DTIM period = 1
    I (9071) esp_netif_handlers: sta ip: 192.168.xx.xx, mask: 255.255.255.0, gw: 192.168.yy.yy
    I (9071) ESP_WIFI_C: wifi station got IP!
    I (107171) wifi:<ba-add>idx:0 (ifx:0, f4:0d:9f:cf:37:40), tid:0, ssn:100, winSize:64