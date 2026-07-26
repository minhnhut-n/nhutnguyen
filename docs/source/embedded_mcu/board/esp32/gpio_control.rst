4. GPIO Control
===============

.. rubric:: Tổng quan

GPIO (General Purpose Input/Output) là một trong những tính năng cơ bản và quan trọng nhất trên ESP32. Bài viết này tổng hợp toàn bộ kiến thức liên quan đến GPIO trên ESP32, bao gồm cấu hình, điều khiển, ngắt, và các ngoại vi liên quan.

.. rubric:: Nội dung

- Cấu hình GPIO cơ bản (Input/Output)
- Pull-up / Pull-down
- Ngắt GPIO (Interrupt)
- PWM với LEDC
- ADC (Analog-to-Digital Converter)
- DAC (Digital-to-Analog Converter)
- Touch Sensor
- RTC GPIO (ngủ sâu)
- GPIO Matrix và IOMUX
- Ví dụ tổng hợp

.. rubric:: 1. Cấu hình GPIO cơ bản

ESP32 có tổng cộng **34 chân GPIO** (GPIO0 - GPIO39), được chia thành các bank:

- **GPIO0 - GPIO19**: Digital I/O, có hỗ trợ RTC, Touch, ADC, ...
- **GPIO21 - GPIO23**: Digital I/O
- **GPIO25 - GPIO27**: Digital I/O, hỗ trợ DAC (GPIO25, GPIO26)
- **GPIO32 - GPIO39**: Digital I/O, GPIO32-39 là input-only (không có pull-up/pull-down)

.. warning::

   Một số chân GPIO có chức năng đặc biệt cần lưu ý:

   - **GPIO0**: Boot mode (kéo LOW khi boot để vào flash mode)
   - **GPIO2**: Boot log (kéo HIGH khi boot)
   - **GPIO5**: VSPI_CS (có thể ảnh hưởng boot)
   - **GPIO12**: Voltage detect (MTDI, kéo HIGH = 1.8V flash)
   - **GPIO15**: MTDO (RTSN, kéo LOW khi boot)

.. rubric:: 1.1. Cấu hình GPIO Output

.. code-block:: c

    #include "driver/gpio.h"

    #define GPIO_LED_PIN    GPIO_NUM_2
    #define GPIO_OUTPUT_PIN_SEL  (1ULL << GPIO_LED_PIN)

    void gpio_output_config(void) {
        gpio_config_t io_conf = {
            .pin_bit_mask = GPIO_OUTPUT_PIN_SEL,
            .mode = GPIO_MODE_OUTPUT,
            .pull_up_en = GPIO_PULLUP_DISABLE,
            .pull_down_en = GPIO_PULLDOWN_DISABLE,
            .intr_type = GPIO_INTR_DISABLE
        };
        gpio_config(&io_conf);
    }

    void gpio_set_high(void) {
        gpio_set_level(GPIO_LED_PIN, 1);
    }

    void gpio_set_low(void) {
        gpio_set_level(GPIO_LED_PIN, 0);
    }

    void gpio_toggle(void) {
        int level = gpio_get_level(GPIO_LED_PIN);
        gpio_set_level(GPIO_LED_PIN, !level);
    }

.. rubric:: 1.2. Cấu hình GPIO Input

.. code-block:: c

    #include "driver/gpio.h"

    #define GPIO_BUTTON_PIN  GPIO_NUM_0
    #define GPIO_INPUT_PIN_SEL  (1ULL << GPIO_BUTTON_PIN)

    void gpio_input_config(void) {
        gpio_config_t io_conf = {
            .pin_bit_mask = GPIO_INPUT_PIN_SEL,
            .mode = GPIO_MODE_INPUT,
            .pull_up_en = GPIO_PULLUP_ENABLE,   // Bật pull-up nội
            .pull_down_en = GPIO_PULLDOWN_DISABLE,
            .intr_type = GPIO_INTR_DISABLE
        };
        gpio_config(&io_conf);
    }

    int read_button(void) {
        return gpio_get_level(GPIO_BUTTON_PIN);
    }

.. rubric:: 2. Pull-up / Pull-down

ESP32 có điện trở pull-up và pull-down nội tích hợp sẵn trên hầu hết các chân GPIO (ngoại trừ GPIO34-39).

.. list-table:: Trạng thái pull-up/pull-down mặc định khi boot
   :header-rows: 1

   * - GPIO
     - Trạng thái
     - Ghi chú
   * - GPIO0
     - Pull-up
     - Boot mode
   * - GPIO2
     - Pull-down
     - Boot log
   * - GPIO5
     - Pull-up
     - VSPI_CS
   * - GPIO12
     - Pull-down
     - Voltage detect
   * - GPIO15
     - Pull-up
     - MTDO

.. code-block:: c

    // Cấu hình pull-up
    gpio_config_t io_conf = {
        .pin_bit_mask = PIN_BIT_MASK,
        .mode = GPIO_MODE_INPUT,
        .pull_up_en = GPIO_PULLUP_ENABLE,
        .pull_down_en = GPIO_PULLDOWN_DISABLE,
        .intr_type = GPIO_INTR_DISABLE
    };
    gpio_config(&io_conf);

    // Hoặc set pull-up sau khi cấu hình
    gpio_set_pull_mode(GPIO_NUM_0, GPIO_PULLUP_ONLY);

.. rubric:: 3. Ngắt GPIO (Interrupt)

ESP32 hỗ trợ 4 loại ngắt GPIO:

- ``GPIO_INTR_POSEDGE``: Ngắt cạnh lên
- ``GPIO_INTR_NEGEDGE``: Ngắt cạnh xuống
- ``GPIO_INTR_ANYEDGE``: Ngắt cả 2 cạnh
- ``GPIO_INTR_LOW_LEVEL``: Ngắt mức thấp
- ``GPIO_INTR_HIGH_LEVEL``: Ngắt mức cao

.. code-block:: c

    #include "driver/gpio.h"

    #define GPIO_BUTTON_PIN  GPIO_NUM_0
    #define GPIO_INPUT_PIN_SEL  (1ULL << GPIO_BUTTON_PIN)

    // Hàm xử lý ngắt (ISR) - phải nhanh, không dùng delay
    static void IRAM_ATTR gpio_isr_handler(void* arg) {
        uint32_t gpio_num = (uint32_t) arg;
        // Xử lý ngắt tại đây (tối giản)
        // Nên dùng queue để gửi event về task chính
    }

    void gpio_interrupt_config(void) {
        // Cấu hình GPIO input
        gpio_config_t io_conf = {
            .pin_bit_mask = GPIO_INPUT_PIN_SEL,
            .mode = GPIO_MODE_INPUT,
            .pull_up_en = GPIO_PULLUP_ENABLE,
            .pull_down_en = GPIO_PULLDOWN_DISABLE,
            .intr_type = GPIO_INTR_NEGEDGE  // Ngắt cạnh xuống (nút nhấn)
        };
        gpio_config(&io_conf);

        // Cài đặt ISR service
        gpio_install_isr_service(ESP_INTR_FLAG_DEFAULT);

        // Đăng ký handler cho GPIO
        gpio_isr_handler_add(GPIO_BUTTON_PIN, gpio_isr_handler, (void*) GPIO_BUTTON_PIN);
    }

.. rubric:: 3.1. Xử lý ngắt với Queue (khuyến nghị)

.. code-block:: c

    #include "freertos/FreeRTOS.h"
    #include "freertos/task.h"
    #include "freertos/queue.h"
    #include "driver/gpio.h"

    static QueueHandle_t gpio_evt_queue = NULL;

    // ISR handler - chỉ gửi event vào queue
    static void IRAM_ATTR gpio_isr_handler(void* arg) {
        uint32_t gpio_num = (uint32_t) arg;
        xQueueSendFromISR(gpio_evt_queue, &gpio_num, NULL);
    }

    // Task xử lý ngắt - chạy trong context task bình thường
    static void gpio_task_example(void* arg) {
        uint32_t io_num;
        while (1) {
            if (xQueueReceive(gpio_evt_queue, &io_num, portMAX_DELAY)) {
                printf("GPIO %d pressed!\n", io_num);
                gpio_set_level(GPIO_LED_PIN, gpio_get_level(GPIO_LED_PIN) ^ 1);
            }
        }
    }

    void app_main(void) {
        // Cấu hình GPIO
        gpio_interrupt_config();

        // Tạo queue
        gpio_evt_queue = xQueueCreate(10, sizeof(uint32_t));

        // Tạo task xử lý ngắt
        xTaskCreate(gpio_task_example, "gpio_task_example", 2048, NULL, 10, NULL);
    }

.. rubric:: 4. PWM với LEDC

ESP32 có module **LEDC (LED Control)** để tạo tín hiệu PWM với độ phân giải lên đến 16-bit.

.. rubric:: 4.1. Cấu hình LEDC cơ bản

.. code-block:: c

    #include "driver/ledc.h"

    #define LEDC_GPIO        GPIO_NUM_2
    #define LEDC_CHANNEL     LEDC_CHANNEL_0
    #define LEDC_TIMER       LEDC_TIMER_0
    #define LEDC_MODE        LEDC_LOW_SPEED_MODE
    #define LEDC_DUTY_RES    LEDC_TIMER_13_BIT   // Độ phân giải 13-bit (0-8191)
    #define LEDC_FREQ_HZ     5000                // Tần số PWM 5kHz

    void ledc_pwm_init(void) {
        // Cấu hình timer
        ledc_timer_config_t ledc_timer = {
            .speed_mode       = LEDC_MODE,
            .timer_num        = LEDC_TIMER,
            .duty_resolution  = LEDC_DUTY_RES,
            .freq_hz          = LEDC_FREQ_HZ,
            .clk_cfg          = LEDC_AUTO_CLK
        };
        ledc_timer_config(&ledc_timer);

        // Cấu hình channel
        ledc_channel_config_t ledc_channel = {
            .channel    = LEDC_CHANNEL,
            .duty       = 0,
            .gpio_num   = LEDC_GPIO,
            .speed_mode = LEDC_MODE,
            .hpoint     = 0,
            .timer_sel  = LEDC_TIMER
        };
        ledc_channel_config(&ledc_channel);
    }

    // Set duty cycle (0 - 8191 cho 13-bit)
    void ledc_set_duty_value(uint32_t duty) {
        ledc_set_duty(LEDC_MODE, LEDC_CHANNEL, duty);
        ledc_update_duty(LEDC_MODE, LEDC_CHANNEL);
    }

    // Fade LED (tăng/giảm dần độ sáng)
    void ledc_fade_led(void) {
        ledc_set_fade_with_time(LEDC_MODE, LEDC_CHANNEL, 4096, 1000);
        ledc_fade_start(LEDC_MODE, LEDC_CHANNEL, LEDC_FADE_NO_WAIT);
    }

.. rubric:: 4.2. Ví dụ LED nhấp nháy với PWM

.. code-block:: c

    void app_main(void) {
        ledc_pwm_init();

        while (1) {
            // Tăng dần độ sáng
            for (int duty = 0; duty <= 8191; duty += 100) {
                ledc_set_duty_value(duty);
                vTaskDelay(pdMS_TO_TICKS(10));
            }
            // Giảm dần độ sáng
            for (int duty = 8191; duty >= 0; duty -= 100) {
                ledc_set_duty_value(duty);
                vTaskDelay(pdMS_TO_TICKS(10));
            }
        }
    }

.. rubric:: 5. ADC (Analog-to-Digital Converter)

ESP32 có 2 bộ ADC (SAR ADC) với độ phân giải 12-bit (có thể cấu hình 9-12 bit).

.. list-table:: ADC Channel và GPIO tương ứng
   :header-rows: 1

   * - ADC Unit
     - Channel
     - GPIO
   * - ADC1
     - CH0
     - GPIO36
   * - ADC1
     - CH3
     - GPIO39
   * - ADC1
     - CH4
     - GPIO32
   * - ADC1
     - CH5
     - GPIO33
   * - ADC1
     - CH6
     - GPIO34
   * - ADC1
     - CH7
     - GPIO35
   * - ADC2
     - CH0
     - GPIO4
   * - ADC2
     - CH1
     - GPIO0
   * - ADC2
     - CH2
     - GPIO2
   * - ADC2
     - CH3
     - GPIO15
   * - ADC2
     - CH4
     - GPIO13
   * - ADC2
     - CH5
     - GPIO12
   * - ADC2
     - CH6
     - GPIO14
   * - ADC2
     - CH7
     - GPIO27
   * - ADC2
     - CH8
     - GPIO25
   * - ADC2
     - CH9
     - GPIO26

.. warning::

   **ADC2** không thể sử dụng đồng thời với Wi-Fi. Khi Wi-Fi đang hoạt động, chỉ nên dùng ADC1.

.. code-block:: c

    #include "driver/adc.h"
    #include "esp_adc_cal.h"

    #define ADC_GPIO        GPIO_NUM_34
    #define ADC_CHANNEL     ADC1_CHANNEL_6   // GPIO34 = ADC1_CH6
    #define ADC_ATTEN       ADC_ATTEN_DB_11  // Đo được 0-3.3V
    #define ADC_WIDTH       ADC_WIDTH_BIT_12 // Độ phân giải 12-bit

    static esp_adc_cal_characteristics_t adc_chars;

    void adc_init(void) {
        // Cấu hình ADC
        adc1_config_width(ADC_WIDTH);
        adc1_config_channel_atten(ADC_CHANNEL, ADC_ATTEN);

        // Hiệu chuẩn ADC
        esp_adc_cal_characterize(ADC_UNIT_1, ADC_ATTEN, ADC_WIDTH, 0, &adc_chars);
    }

    uint32_t adc_read_raw(void) {
        return adc1_get_raw(ADC_CHANNEL);
    }

    uint32_t adc_read_voltage(void) {
        uint32_t raw = adc1_get_raw(ADC_CHANNEL);
        uint32_t voltage = 0;
        esp_adc_cal_get_voltage(ADC_CHANNEL, &adc_chars, &voltage);
        return voltage;  // mV
    }

.. rubric:: 5.1. Đọc ADC với attenuation

.. list-table:: Attenuation và dải đo
   :header-rows: 1

   * - Attenuation
     - Dải đo
   * - ADC_ATTEN_DB_0
     - 0 - 1.1V
   * - ADC_ATTEN_DB_2_5
     - 0 - 1.5V
   * - ADC_ATTEN_DB_6
     - 0 - 2.2V
   * - ADC_ATTEN_DB_11
     - 0 - 3.3V

.. rubric:: 6. DAC (Digital-to-Analog Converter)

ESP32 có 2 kênh DAC 8-bit trên GPIO25 (DAC1) và GPIO26 (DAC2).

.. code-block:: c

    #include "driver/dac.h"

    void dac_init(void) {
        // Khởi tạo DAC trên GPIO25
        dac_output_enable(DAC_CHANNEL_1);
    }

    void dac_set_voltage(uint8_t voltage) {
        // voltage: 0-255 (0V - 3.3V)
        dac_output_voltage(DAC_CHANNEL_1, voltage);
    }

    // Tạo sóng sin bằng DAC
    void dac_sine_wave(void) {
        uint8_t sine_wave[256];
        for (int i = 0; i < 256; i++) {
            sine_wave[i] = (uint8_t)(128 + 127 * sin(2 * M_PI * i / 256));
        }

        while (1) {
            for (int i = 0; i < 256; i++) {
                dac_output_voltage(DAC_CHANNEL_1, sine_wave[i]);
                ets_delay_us(50);  // ~78Hz
            }
        }
    }

.. rubric:: 7. Touch Sensor

ESP32 có 10 cảm biến touch điện dung, hoạt động dựa trên nguyên lý đo thời gian nạp/xả tụ điện.

.. list-table:: Touch channel và GPIO
   :header-rows: 1

   * - Touch Channel
     - GPIO
   * - T0
     - GPIO4
   * - T1
     - GPIO0
   * - T2
     - GPIO2
   * - T3
     - GPIO15
   * - T4
     - GPIO13
   * - T5
     - GPIO12
   * - T6
     - GPIO14
   * - T7
     - GPIO27
   * - T8
     - GPIO33
   * - T9
     - GPIO32

.. code-block:: c

    #include "driver/touch_pad.h"

    #define TOUCH_GPIO      GPIO_NUM_4
    #define TOUCH_CHANNEL   TOUCH_PAD_NUM0   // T0 = GPIO4

    void touch_sensor_init(void) {
        touch_pad_init();
        touch_pad_config(TOUCH_CHANNEL);
        touch_pad_set_voltage(TOUCH_HVOLT_2V7, TOUCH_LVOLT_0V5, TOUCH_HVOLT_ATTEN_1V);
        touch_pad_set_meas_time(1000);
    }

    uint32_t touch_sensor_read(void) {
        uint32_t touch_value;
        touch_pad_read(TOUCH_CHANNEL, &touch_value);
        return touch_value;  // Giá trị càng nhỏ khi chạm
    }

    // Touch với ngắt
    void touch_sensor_interrupt_init(void) {
        touch_pad_init();
        touch_pad_config(TOUCH_CHANNEL);

        // Set ngưỡng (threshold) cho touch
        uint32_t touch_value;
        touch_pad_read(TOUCH_CHANNEL, &touch_value);
        touch_pad_set_thresh(TOUCH_CHANNEL, touch_value - 100);

        // Cấu hình ngắt
        touch_pad_isr_register(touch_isr_callback, NULL);
        touch_pad_intr_enable();
    }

.. rubric:: 8. RTC GPIO (Ngủ sâu)

Khi ESP32 ở chế độ **Deep Sleep**, các GPIO thông thường không hoạt động. Tuy nhiên, một số GPIO thuộc **RTC domain** có thể được sử dụng để đánh thức chip hoặc duy trì trạng thái.

.. list-table:: RTC GPIO
   :header-rows: 1

   * - RTC GPIO
     - GPIO tương ứng
   * - RTC_GPIO0
     - GPIO0
   * - RTC_GPIO1
     - GPIO2
   * - RTC_GPIO2
     - GPIO4
   * - RTC_GPIO3
     - GPIO12
   * - RTC_GPIO4
     - GPIO13
   * - RTC_GPIO5
     - GPIO14
   * - RTC_GPIO6
     - GPIO15
   * - RTC_GPIO7
     - GPIO25
   * - RTC_GPIO8
     - GPIO26
   * - RTC_GPIO9
     - GPIO27
   * - RTC_GPIO10
     - GPIO32
   * - RTC_GPIO11
     - GPIO33

.. code-block:: c

    #include "driver/rtc_io.h"
    #include "esp_sleep.h"

    #define WAKEUP_GPIO     GPIO_NUM_4   // RTC_GPIO2

    void deep_sleep_setup(void) {
        // Cấu hình RTC GPIO làm input với pull-up
        rtc_gpio_init(WAKEUP_GPIO);
        rtc_gpio_set_direction(WAKEUP_GPIO, RTC_GPIO_MODE_INPUT_ONLY);
        rtc_gpio_pullup_en(WAKEUP_GPIO);
        rtc_gpio_pulldown_dis(WAKEUP_GPIO);

        // Cấu hình đánh thức từ Deep Sleep bằng GPIO (cạnh xuống)
        esp_sleep_enable_ext0_wakeup(WAKEUP_GPIO, 0);
    }

    void go_to_deep_sleep(void) {
        printf("Entering deep sleep...\n");
        esp_deep_sleep_start();
    }

    // Kiểm tra nguyên nhân đánh thức
    void check_wakeup_cause(void) {
        esp_sleep_wakeup_cause_t cause = esp_sleep_get_wakeup_cause();
        switch (cause) {
            case ESP_SLEEP_WAKEUP_EXT0:
                printf("Wakeup from GPIO (EXT0)\n");
                break;
            case ESP_SLEEP_WAKEUP_EXT1:
                printf("Wakeup from GPIO (EXT1)\n");
                break;
            case ESP_SLEEP_WAKEUP_TIMER:
                printf("Wakeup from timer\n");
                break;
            default:
                printf("Not a deep sleep wakeup\n");
                break;
        }
    }

.. rubric:: 8.1. Đánh thức từ nhiều GPIO (EXT1)

.. code-block:: c

    #define WAKEUP_PIN_BITMASK  ((1ULL << GPIO_NUM_4) | (1ULL << GPIO_NUM_12))

    void deep_sleep_ext1_setup(void) {
        // Cấu hình đánh thức từ nhiều GPIO
        // ESP_EXT1_WAKEUP_ANY_LOW: bất kỳ chân nào ở mức thấp
        // ESP_EXT1_WAKEUP_ANY_HIGH: bất kỳ chân nào ở mức cao
        esp_sleep_enable_ext1_wakeup(WAKEUP_PIN_BITMASK, ESP_EXT1_WAKEUP_ANY_LOW);
    }

    // Kiểm tra GPIO nào đã đánh thức
    void check_wakeup_gpio(void) {
        uint64_t wakeup_pin = esp_sleep_get_ext1_wakeup_status();
        if (wakeup_pin & (1ULL << GPIO_NUM_4)) {
            printf("Wakeup from GPIO4\n");
        }
        if (wakeup_pin & (1ULL << GPIO_NUM_12)) {
            printf("Wakeup from GPIO12\n");
        }
    }

.. rubric:: 9. GPIO Matrix và IOMUX

ESP32 có kiến trúc **GPIO Matrix** và **IOMUX** linh hoạt, cho phép ánh xạ hầu hết các tín hiệu ngoại vi đến bất kỳ chân GPIO nào.

.. rubric:: 9.1. IOMUX (Direct Connection)

IOMUX cung cấp kết nối trực tiếp giữa ngoại vi và GPIO, cho độ trễ thấp nhất. Chỉ một số GPIO nhất định có IOMUX cho từng ngoại vi.

.. list-table:: IOMUX mặc định cho UART0
   :header-rows: 1

   * - Tín hiệu
     - GPIO
   * - U0TXD
     - GPIO1
   * - U0RXD
     - GPIO3

.. rubric:: 9.2. GPIO Matrix (Flexible Connection)

GPIO Matrix cho phép ánh xạ bất kỳ tín hiệu ngoại vi nào đến bất kỳ GPIO nào (có độ trễ cao hơn IOMUX một chút).

.. code-block:: c

    #include "soc/gpio_sig_map.h"
    #include "driver/periph_ctrl.h"

    // Ví dụ: ánh xạ UART TX sang GPIO5 thay vì GPIO1 mặc định
    void gpio_matrix_example(void) {
        // Cấu hình GPIO5 làm output
        gpio_set_direction(GPIO_NUM_5, GPIO_MODE_OUTPUT);

        // Kết nối tín hiệu UART TX đến GPIO5 qua GPIO Matrix
        gpio_matrix_out(GPIO_NUM_5, U0TXD_OUT_IDX, false, false);
    }

.. rubric:: 10. Ví dụ tổng hợp

Ví dụ kết hợp nhiều tính năng GPIO: đọc nút nhấn (ngắt), điều khiển LED (PWM), đọc cảm biến (ADC), và touch sensor.

.. code-block:: c

    #include <stdio.h>
    #include "freertos/FreeRTOS.h"
    #include "freertos/task.h"
    #include "freertos/queue.h"
    #include "driver/gpio.h"
    #include "driver/ledc.h"
    #include "driver/adc.h"
    #include "esp_adc_cal.h"
    #include "driver/touch_pad.h"

    // Định nghĩa chân
    #define LED_PIN         GPIO_NUM_2
    #define BUTTON_PIN      GPIO_NUM_0
    #define ADC_PIN         GPIO_NUM_34
    #define TOUCH_PIN       GPIO_NUM_4

    // LEDC config
    #define LEDC_CH         LEDC_CHANNEL_0
    #define LEDC_TIM        LEDC_TIMER_0
    #define LEDC_MODE       LEDC_LOW_SPEED_MODE

    // Queue cho ISR
    static QueueHandle_t gpio_evt_queue = NULL;

    // ADC calibration
    static esp_adc_cal_characteristics_t adc_chars;

    // ISR handler
    static void IRAM_ATTR gpio_isr_handler(void* arg) {
        uint32_t gpio_num = (uint32_t) arg;
        xQueueSendFromISR(gpio_evt_queue, &gpio_num, NULL);
    }

    // Cấu hình LED PWM
    void led_init(void) {
        ledc_timer_config_t ledc_timer = {
            .speed_mode       = LEDC_MODE,
            .timer_num        = LEDC_TIM,
            .duty_resolution  = LEDC_TIMER_13_BIT,
            .freq_hz          = 5000,
            .clk_cfg          = LEDC_AUTO_CLK
        };
        ledc_timer_config(&ledc_timer);

        ledc_channel_config_t ledc_channel = {
            .channel    = LEDC_CH,
            .duty       = 0,
            .gpio_num   = LED_PIN,
            .speed_mode = LEDC_MODE,
            .hpoint     = 0,
            .timer_sel  = LEDC_TIM
        };
        ledc_channel_config(&ledc_channel);
    }

    // Cấu hình nút nhấn với ngắt
    void button_init(void) {
        gpio_config_t io_conf = {
            .pin_bit_mask = (1ULL << BUTTON_PIN),
            .mode = GPIO_MODE_INPUT,
            .pull_up_en = GPIO_PULLUP_ENABLE,
            .pull_down_en = GPIO_PULLDOWN_DISABLE,
            .intr_type = GPIO_INTR_NEGEDGE
        };
        gpio_config(&io_conf);

        gpio_install_isr_service(ESP_INTR_FLAG_DEFAULT);
        gpio_isr_handler_add(BUTTON_PIN, gpio_isr_handler, (void*) BUTTON_PIN);
    }

    // Cấu hình ADC
    void adc_init(void) {
        adc1_config_width(ADC_WIDTH_BIT_12);
        adc1_config_channel_atten(ADC1_CHANNEL_6, ADC_ATTEN_DB_11);
        esp_adc_cal_characterize(ADC_UNIT_1, ADC_ATTEN_DB_11, ADC_WIDTH_BIT_12, 0, &adc_chars);
    }

    // Cấu hình touch
    void touch_init(void) {
        touch_pad_init();
        touch_pad_config(TOUCH_PAD_NUM0);
    }

    // Task chính
    void app_main(void) {
        // Khởi tạo
        led_init();
        button_init();
        adc_init();
        touch_init();

        // Tạo queue
        gpio_evt_queue = xQueueCreate(10, sizeof(uint32_t));

        uint32_t duty = 0;
        bool duty_dir = true;  // true: tăng, false: giảm

        while (1) {
            // Đọc nút nhấn (non-blocking)
            uint32_t io_num;
            if (xQueueReceive(gpio_evt_queue, &io_num, 0)) {
                printf("Button %d pressed!\n", io_num);
                duty_dir = !duty_dir;
            }

            // PWM LED
            ledc_set_duty(LEDC_MODE, LEDC_CH, duty);
            ledc_update_duty(LEDC_MODE, LEDC_CH);

            if (duty_dir) {
                duty += 100;
                if (duty >= 8191) duty_dir = false;
            } else {
                duty -= 100;
                if (duty <= 0) duty_dir = true;
            }

            // Đọc ADC
            uint32_t adc_voltage = 0;
            esp_adc_cal_get_voltage(ADC1_CHANNEL_6, &adc_chars, &adc_voltage);
            printf("ADC Voltage: %d mV\n", adc_voltage);

            // Đọc touch
            uint32_t touch_val;
            touch_pad_read(TOUCH_PAD_NUM0, &touch_val);
            printf("Touch value: %d\n", touch_val);

            vTaskDelay(pdMS_TO_TICKS(100));
        }
    }

.. rubric:: Tài liệu tham khảo

- `ESP32 Technical Reference Manual <https://www.espressif.com/sites/default/files/documentation/esp32_technical_reference_manual_en.pdf>`_ - Chapter 4: GPIO & Chapter 28: IO MUX
- `ESP-IDF GPIO Driver Guide <https://docs.espressif.com/projects/esp-idf/en/latest/esp32/api-reference/peripherals/gpio.html>`_
- `ESP-IDF LEDC Driver Guide <https://docs.espressif.com/projects/esp-idf/en/latest/esp32/api-reference/peripherals/ledc.html>`_
- `ESP-IDF ADC Driver Guide <https://docs.espressif.com/projects/esp-idf/en/latest/esp32/api-reference/peripherals/adc.html>`_
- `ESP32 Pinout Reference <https://docs.espressif.com/projects/esp-idf/en/latest/esp32/_images/esp32-devkitC-v4-pinout.png>`_