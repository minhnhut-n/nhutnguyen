I. Software Architecture: From theory to practice
=============================================================================

.. rubric:: 1. Giới Thiệu Về Architecture Trong Xây Dựng Một Mã Nguồn

Trong kỹ nghệ phần mềm, Kiến trúc phần mềm (Software Architecture) đóng vai trò như bản thiết kế tổng thể định hình cấu trúc, mối quan hệ tương tác và các quy tắc tổ chức giữa các thành phần phần cứng và phần mềm. Kiến trúc không đơn thuần là việc phân chia mã nguồn thành các thư mục hay hàm nhỏ, mà là việc đưa ra các quyết định chiến lược mang tính cốt lõi rất khó thay đổi sau này.

Tương tự như xây dựng một tòa nhà, nếu phần móng và bộ khung không vững chắc hoặc thiết kế sai mục đích (như xây nhà dân dụng nhưng muốn chịu tải như nhà xưởng), việc thêm bớt các tầng hay thay đổi kết cấu về sau sẽ vô cùng đắt đỏ và tiềm ẩn nguy cơ đổ vỡ. Một kiến trúc chuẩn mực giúp định hình:

- **Sự phân chia trách nhiệm (Separation of Concerns):** Mỗi phần tử trong hệ thống chỉ đảm nhận một công việc rõ ràng.
- **Dòng luồng dữ liệu & kiểm soát (Control & Data Flow):** Quy định cách thức các module giao tiếp, trao đổi thông điệp với nhau.
- **Sự quản lý tài nguyên (Resource Management):** Tối ưu hóa việc sử dụng CPU, RAM, bộ nhớ Flash và Bus giao tiếp, đặc biệt quan trọng trong các môi trường giới hạn như vi điều khiển (MCU).

**Mở rộng chức năng (Scalability & Extensibility)**

- **Thiết Kế Tệ:** Mã nguồn dạng "Bún xào" (Spaghetti Code): Mọi tính năng mới đòi hỏi phải sửa đổi sâu vào mã nguồn hiện tại. Sửa một chỗ gây gãy lỗi ở nhiều chỗ không liên quan.
- **Thiết Kế Tốt:** Tính mở rộng mô-đun (Plug-and-Play): Áp dụng nguyên lý OCP (Open/Closed Principle). Thêm tính năng mới bằng cách bổ sung module mới mà không làm ảnh hưởng hay phải biên dịch lại mã nguồn cũ.

**Tranh chấp tài nguyên & Sự kiện (Contention)**

- **Thiết Kế Tệ:** Xung đột & Nghẽn (Blocking & Deadlocks): Mã nguồn viết theo kiểu tuần tự (Sequence) khiến một task mất nhiều thời gian sẽ khóa toàn bộ CPU, gây trễ ngắt, trôi timer và treo các sự kiện quan trọng.
- **Thiết Kế Tốt:** Bất đồng bộ & Lập lịch (Asynchronous & RTOS Scheduling): Các handler xử lý gọn nhẹ, giải phóng CPU ngay lập tức. Tận dụng RTOS scheduler hoặc Event Loop để điều phối tài nguyên mượt mà.

**Luồng công việc (Workflow Smoothness)**

- **Thiết Kế Tệ:** Phụ thuộc chồng chéo (Cyclic Dependency): Các file #include hoặc import lẫn nhau, tạo ra sự phụ thuộc chặt chẽ (High Coupling), rất khó kiểm soát luồng thực thi và bộ nhớ.
- **Thiết Kế Tốt:** Giảm phụ thuộc (Loose Coupling): Luồng dữ liệu rõ ràng thông qua interface, callback hoặc bus sự kiện. Giúp hệ thống hoạt động ổn định và mượt mà.

**Bảo trì & Kiểm thử (Testability & Maintainability)**

- **Thiết Kế Tệ:** Không thể viết Unit Test độc lập. Bất kỳ thử nghiệm nào cũng đòi hỏi phải nạp lên toàn bộ phần cứng thực tế hoặc dựng toàn bộ hệ thống server.
- **Thiết Kế Tốt:** Có thể Unit Test độc lập từng phần bằng Mock/Stub. Dễ dàng debug và phát hiện lỗi sớm ngay trên môi trường giả lập.

.. rubric:: 3. Các Thiết Kế Từ Tốt Đến Tệ, Cùng Ưu Nhược Điểm

**1. Modular Architecture / Vertical Slice (Cắt Dọc) — [TỐT NHẤT]**

- **Mô tả:** Hệ thống được chia theo từng tính năng/nghiệp vụ hoàn chỉnh (Vertical Feature Slices) từ giao diện, logic đến lưu trữ dữ liệu, thay vì chia theo lớp kỹ thuật.
- **Ưu điểm:** Tính độc lập cực cao. Khi thay đổi một tính năng, engineer chỉ cần thao tác trên slice đó mà không sợ chạm vào các slice khác. Rất dễ gỡ bỏ hoặc đóng gói module để tái sử dụng.
- **Nhược điểm:** Có thể dẫn đến lặp lại một số mã nguồn dùng chung nếu không tổ chức shared library hợp lý.

**2. Event-Driven / Reactive Architecture — [RẤT TỐT]**

- **Mô tả:** Các module tương tác hoàn toàn thông qua việc phát (Publish) và lắng nghe (Subscribe) các sự kiện (Events).
- **Ưu điểm:** Bất đồng bộ hóa triệt để, không blocking CPU. Các module hoàn toàn không biết sự tồn tại của nhau (Decoupled), giúp mở rộng hệ thống cực kỳ dễ dàng.
- **Nhược điểm:** Luồng thực thi khó theo dõi hơn khi debug step-by-step; đòi hỏi quản lý bộ nhớ event-queue cẩn thận để tránh tràn RAM.

**3. Layered / N-Tier Architecture (Layered Cổ Điển) — [TRUNG BÌNH]**

- **Mô tả:** Chia phần mềm thành các tầng nằm ngang: Presentation Layer → Business Logic Layer → Hardware/Data Access Layer. Tầng trên chỉ được gọi xuống tầng dưới.
- **Ưu điểm:** Dễ hiểu, cấu trúc rõ ràng, phù hợp với các ứng dụng quy mô vừa và nhỏ.
- **Nhược điểm:** Rất dễ bị phình to lớp giữa (Anemic Domain / Sinkhole Pattern); khi thay đổi một trường dữ liệu nhỏ phải sửa qua tất cả các tầng.

**4. Monolithic (Tập Trung Đơn Khối) — [TỆ / HẠN CHẾ]**

- **Mô tả:** Tất cả mã nguồn, xử lý logic, ngắt hardware và giao diện gom chung vào một vài tập tin chính (hoặc thậm chí 1 file main.c duy nhất).
- **Ưu điểm:** Rất nhanh khi triển khai các bài test nhỏ (Proof of Concept), không tốn chi phí tổ chức abstractions.
- **Nhược điểm:** Không thể bảo trì khi dự án lớn dần. Xảy ra xung đột chồng chéo ngắt, khó làm việc nhóm và gần như không thể tái sử dụng.

.. rubric:: 4. Phân Tích Chi Tiết Các Thiết Kế: Event-Driven, Cross-Cutting / Vertical Slice

**A. Mô Hình Hướng Sự Kiện (Event-Driven Architecture)**

Thay vì gọi trực tiếp hàm xử lý (Synchronous Function Call), module phát sự kiện chỉ tạo một tin nhắn event và đẩy vào Event Queue / Bus. Module xử lý sẽ nhận event này từ queue và xử lý bất đồng bộ.

::

  [Event Source: Sensor Interrupt / User Button]
         │
         ▼ (Emit Event & Push to Queue - Non-blocking)
  [ Central Event Broker / Dispatcher Queue ]
         │
         ▼ (Dispatch Asynchronously)
  [ Task / Handler A ] │ [ Task / Handler B ] │ [ Task / Handler C ]

**Ví dụ cụ thể trên C/RTOS (ESP32 Multi-core):**

Nút bấm bị nhấn (ISR). Thay vì xử lý gửi tin nhắn WiFi ngay trong ISR hay trong hàm loop chính:

.. code-block:: c

  // Mã nguồn ví dụ Event-Driven với FreeRTOS trên ESP32
  typedef enum {
      EVENT_BUTTON_PRESSED,
      EVENT_SENSOR_DATA_READY
  } SystemEvent_t;

  QueueHandle_t systemEventQueue;

  // Interrupt Service Routine (ISR) - Giải phóng CPU ngay lập tức
  void IRAM_ATTR button_isr_handler() {
      SystemEvent_t evt = EVENT_BUTTON_PRESSED;
      xQueueSendFromISR(systemEventQueue, &evt, NULL);
  }

  // Network Task chạy bất đồng bộ trên Core 1
  void vNetworkTask(void *pvParameters) {
      SystemEvent_t evt;
      for(;;) {
          if (xQueueReceive(systemEventQueue, &evt, portMAX_DELAY)) {
              if (evt == EVENT_BUTTON_PRESSED) {
                  // Xử lý gửi HTTP Request/MQTT tại đây mà không block Core 0
                  send_mqtt_message("Button Triggered!");
              }
          }
      }
  }

**B. Mô Hình Cắt Dọc & Cross-Cutting Concerns (Vertical Slice / Aspect-Oriented)**

Vertical Slice Architecture: Phân chia ứng dụng theo từng Chức Năng (Use Case). Ví dụ trong ứng dụng nhà thông minh: Module Bật/Tắt Đèn, Module Đo Nhiệt Độ, Module Cập Nhật OTA. Mỗi module chứa đầy đủ từ giao diện/sensor hardware cho đến logic xử lý và lưu RAM/Flash riêng.

Cross-Cutting Concerns (Các vấn đề dùng chung xuyên suốt): Là các tính năng mà tất cả các Vertical Slices đều cần đến, ví dụ: Logging, Security/Authentication, Power Management, Memory Pooling. Các thành phần này sẽ được thiết kế dưới dạng Middleware hoặc Interceptors để can thiệp vào các slice mà không làm bẩn logic nghiệp vụ chính.

.. code-block:: c

  // Minh họa Cross-Cutting Concerns qua cơ chế Logging/Power Wrapper
  void execute_vertical_feature(void (*feature_func)(void), const char* feature_name) {
      // Cross-Cutting Concern 1: Security/Power Check
      if (!power_manager_is_stable()) return;
      
      // Cross-Cutting Concern 2: Logging
      LOG_INFO("Executing feature: %s", feature_name);
      
      // Thực thi Vertical Slice logic chính
      feature_func();
      
      // Cross-Cutting Concern 3: Execution Metrics
      metrics_record_execution(feature_name);
  }

.. rubric:: 5. Ứng Dụng Mô Hình & Cách Lựa Chọn Cho System Target (MCU vs LINUX)

**A. Tiêu Chí Lựa Chọn Dựa Trên System Target**

**1. Vi Điều Khiển Giới Hạn (Bare-metal MCU: STM32F1, PIC, AVR)**

- **Đặc Điểm & Hạn Chế:** RAM rất nhỏ (vài KB), Single-core, không có OS. Không có bộ quản lý bộ nhớ (MMU). Dễ tràn Stack/Heap nếu cấp phát động.
- **Kiến Trúc Tối Ưu:** FSM (Finite State Machine) + Event Loop (Cooperative Scheduling): Kết hợp với Data-Driven nhẹ. Tránh dùng threading phức tạp, ưu tiên tĩnh hóa bộ nhớ (Static Allocation).

**2. MCU Cao Cấp / Multi-Core (ESP32, STM32H7, NRF5340) với RTOS**

- **Đặc Điểm & Hạn Chế:** Có RTOS (FreeRTOS, Zephyr), đa nhân (Dual-core), hỗ trợ lập lịch Preemptive, tài nguyên RAM từ vài trăm KB đến vài MB.
- **Kiến Trúc Tối Ưu:** Event-Driven + Vertical Slice: Mỗi module nghiệp vụ độc lập chạy trên các Task RTOS khác nhau, giao tiếp qua Queues/Message Buffers. Tận dụng Core 0 cho Wireless Stack (Wi-Fi/Bluetooth) và Core 1 cho App Logic.

**3. Hệ Điều Hành Khái Niệm Mở (Embedded Linux, Raspberry Pi, Server)**

- **Đặc Điểm & Hạn Chế:** Có MMU, RAM lớn (GB), chạy hệ điều hành Đa nhiệm/Đa tiến trình (POSIX, Multi-threading, Virtual Memory).
- **Kiến Trúc Tối Ưu:** Microservices / Modular Vertical Slice + Publish-Subscribe Bus (D-Bus, MQTT, gRPC): Phân chia ứng dụng thành các Process riêng biệt. Rất an toàn vì lỗi ở 1 process không làm sập toàn hệ thống.

**B. Phương Pháp Đánh Giá Lựa Chọn Kiến Trúc (Decision Matrix)**

- **Xác định giới hạn thời gian thực (Hard Real-time vs Soft Real-time):** Nếu là Hard Real-time (như điều khiển động cơ, phanh ABS), ưu tiên Interrupt-Driven + State Machine để latency là cực trị bé nhất.
- **Đánh giá mức độ biến động của yêu cầu (Requirement Turbulence):** Nếu các yêu cầu nghiệp vụ liên tục thay đổi, ưu tiên Vertical Slice để cô lập ảnh hưởng của thay đổi.
- **Đánh giá số lượng kỹ sư cùng phát triển:** Nếu nhóm lớn cùng phát triển trên 1 repository, Vertical Slice giúp giảm thiểu tối đa tình trạng xung đột mã nguồn (Git Merge Conflict).

.. rubric:: 6. Kết Luận: Tổng Quan Về 2 Thiết Kế Trọng Tâm

**Event-Driven Architecture:** Giải quyết triệt để bài toán tranh chấp tài nguyên, nghẽn sự kiện (Contention) và tính phụ thuộc thời gian (Temporal Coupling). Bằng cách chuyển giao tiếp về dạng tin nhắn bất đồng bộ, hệ thống có khả năng đáp ứng nhanh, tận dụng tối đa năng lực xử lý đa nhân của các dòng MCU hiện đại như ESP32 hay hệ thống đa tiến trình trên Linux.

**Vertical Slice Architecture:** Giải quyết bài toán mở rộng tính năng và giảm sự chồng chéo mã nguồn (Spatial Coupling). Bằng cách đóng gói trọn vẹn từng chức năng thành một module riêng biệt, dự án đạt được tính linh hoạt tối đa, cho phép thêm, xóa, sửa các tính năng mà không lo ngại làm ảnh hưởng tới sự ổn định tổng thể.

Không có một kiến trúc nào là "viên đạn bạc" (Silver Bullet) cho mọi bài toán. Việc lựa chọn kiến trúc đòi hỏi sự cân bằng giữa Giới hạn Phần cứng, Phức tạp Nguyện vọng Nghiệp vụ và Năng lực Tái sử dụng Mã nguồn. Việc áp dụng đúng mô hình ngay từ bước phân tích bài toán sẽ tiết kiệm hàng trăm giờ debug và refactoring trong tương lai.

.. rubric:: Tài Liệu Tham Khảo

- `Software Architecture Patterns <https://en.wikipedia.org/wiki/Software_architecture>`_
- `Event-Driven Architecture Guide <https://docs.microsoft.com/en-us/azure/architecture/patterns/event-driven-architecture>`_
- `Clean Architecture by Robert C. Martin <https://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html>`_