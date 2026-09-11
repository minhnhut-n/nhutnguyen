===================
Kernel to Edge AI
===================

:Tác giả: Kỹ sư Performance / Kernel / Edge AI Systems
:Ngày tạo: Tháng 9, 2026
:Trạng thái: Đang thực hiện
:Định hướng cốt lõi: Tận dụng thế mạnh chuyên sâu về Kernel (Scheduler EEVDF/CFS, ZRAM, Memory Compression, Performance/Boost Service) để chuyển hướng thành System / Performance AI Engineer. Tập trung tối ưu hóa mô hình AI, Inference Engine, Latency, Memory và Power trên phần cứng thực tế.

.. contents:: Mục lục
   :depth: 2
   :local:

.. _danh-gia-profile:

1. ĐÁNH GIÁ PROFILE & LỢI THẾ CẠNH TRANH (UNFAIR ADVANTAGE)
=============================================================

Hầu hết kỹ sư AI hiện tại đi lên từ mảng Software/Data Science, giỏi huấn luyện mô hình nhưng thường gặp khó khăn lớn khi triển khai (Deployment) và tối ưu hạ tầng thực thi. Điểm mạnh cốt lõi của bạn nằm ở tầng phần cứng, hệ điều hành và tối ưu hiệu năng.

.. list-table:: Ánh xạ Kỹ năng từ Kernel/System sang Edge AI
   :widths: 35 65
   :header-rows: 1

   * - Kỹ năng Kernel/System hiện tại
     - Ánh xạ sang Kỹ năng Edge AI / AI Systems
   * - **Scheduler (EEVDF / CFS) & Boost Service**
     - **Heterogeneous Task Scheduling:** Phân phối luồng Inference tối ưu giữa CPU - GPU - NPU, kiểm soát Latency/Jitter, cô lập CPU (``isolcpus``), quản lý Thread Affinity và ``SCHED_FIFO`` cho luồng AI thời gian thực.
   * - **ZRAM & Memory Compression**
     - **Memory-bound AI Optimization:** Tối ưu Memory Footprint, Memory Bandwidth, hỗ trợ nén KV Cache, hiểu sâu về cơ chế Quantization (INT8/INT4), Pruning và Memory Allocation.
   * - **System Overview & Root-cause Debugging**
     - **Inference Bottleneck Profiling:** Nhận diện nhanh các điểm nghẽn hiệu năng như CPU Cache Thrashing, Memory Bandwidth Bottlenecks, Thermal Throttling và Latency Spikes.

.. _dieu-chinh-du-an:

2. CHIẾN LƯỢC ĐIỀU CHINH DỰ ÁN & HỌC TẬP (LỌC BỎ LAN MAN)
============================================================

Để khắc phục tình trạng bị quá tải (overwhelmed) và phân tán nguồn lực, hệ thống lại các ý tưởng dự án như sau:

- **[ĐÓNG GÓI & DỪNG] ESP32 Dashboard & RF24 HAL (C, Singleton, Event-driven):**
  Đã hoàn thành xuất sắc. Đóng gói code, viết README đẹp trên GitHub để làm Portfolio chứng minh tư duy C Clean/Design Pattern.
- **[TẠM DỪNG / LỌC BỎ] ESP32-P4 Android Auto:**
  Tốn nhiều thời gian cho phần Integration/Protocol USB Host/Display, chệch hướng khỏi mục tiêu AI Systems.
- **[TẬP TRUNG 100%] Edge AI Translate / Audio Processing System:**
  Dự án đinh kết hợp trọn vẹn giữa Audio Real-time Pipeline, Memory/Latency Optimization và AI Inference.

.. _du-an-dinh:

3. DỰ ÁN ĐINH: EDGE AI AUDIO / TRANSLATE SYSTEM
=================================================

Dự án này là minh chứng rõ nhất cho khả năng kết hợp giữa **System Engineering** và **Edge AI**.

3.1 Kiến trúc Hệ thống 3 Tầng
-------------------------------

1. **Tầng Frontend (Microcontroller / DSP):**
   - Dùng ESP32-S3 / ESP32-P4 thu âm qua I2S Microphone.
   - Tích hợp mô hình TinyML cực nhẹ (TFLite Micro) để làm Voice Activity Detection (VAD) hoặc Keyword Spotting (KWS), lọc nhiễu ban đầu.

2. **Tầng Processing Pipeline (Embedded Linux / Host System):**
   - Đưa luồng Audio qua PipeWire / ALSA.
   - Chạy mô hình Whisper STT đã được nén (INT8 Quantized / ONNX Runtime / ``whisper.cpp``).

3. **Tầng System Tuning (Thế mạnh Kernel của bạn):**
   - **Optimize Latency:** Chỉnh ``SCHED_FIFO``, gán CPU Affinity (Taskset/Isolcpus) cho thread Inference để đạt Zero-latency jitter.
   - **Optimize Memory:** Sử dụng ``mmap``, quản lý Ring Buffer, tối ưu Memory Footprint và nén Cache tương tự tư duy ZRAM.

.. _lo-trinh-5-buoc:

4. LỘ TRÌNH 5 BƯỚC PHÁT TRIỂN TỪ KERNEL SANG EDGE AI
======================================================

4.1 Bước 1: Nắm AI Nền tảng vừa đủ
----------------------------------

- **Công cụ:** Python, NumPy, Pandas.
- **Kiến thức:** Xác suất thống kê cơ bản, Classification, Regression, Overfitting, Metrics (Accuracy, Precision, Recall, Latency).
- **Mục tiêu:** Hiểu bản chất cách mô hình vận hành và đánh giá được chất lượng mô hình.

4.2 Bước 2: Deep Learning cho Tín hiệu & Thị giác
---------------------------------------------------

- **Framework:** PyTorch (Dùng để thử nghiệm và export mô hình).
- **Mô hình:** 1D-CNN, RNN/LSTM cho dữ liệu chuỗi/âm thanh; CNN cho hình ảnh.
- **Thực hành sát Nhúng:** Keyword Spotting (KWS), Gesture Recognition từ IMU, Anomaly Detection cho động cơ.

4.3 Bước 3: Tối ưu & Nén mô hình xuống Thiết bị biên (Edge Conversion)
----------------------------------------------------------------------

- **Công cụ:** ONNX, TensorFlow Lite / TFLite Micro, ``llama.cpp`` / ``whisper.cpp``.
- **Kỹ thuật:** Quantization (INT8 / INT4), Pruning, Graph Fusing.
- **Kỹ năng Đánh đổi (Trade-off):** Cân bằng giữa Accuracy | Latency | RAM/Flash | Power Consumption.

4.4 Bước 4: Triển khai & Tối ưu trên Nền tảng Phần cứng
--------------------------------------------------------

- **MCU Level:** STM32, ESP32-S3/P4 sử dụng TFLite Micro, CMSIS-NN.
- **Embedded Linux / SoC:** Raspberry Pi, NVIDIA Jetson, Orange Pi sử dụng ONNX Runtime, TensorRT, OpenVINO.
- **Hardware Acceleration:** Tương tác với NPU/GPU drivers, SIMD/NEON instructions.

4.5 Bước 5: MLOps cho Thiết bị thực & System Profiling
-------------------------------------------------------

- **System Tools:** Dùng ``perf``, ``gprof``, ``valgrind``, ``ftrace`` để đo đạc nghẽn bộ nhớ/CPU khi Inference.
- **Edge MLOps:** Model Versioning, Secure OTA Update (Cập nhật mô hình an toàn), Data Drift Detection, Privacy.

.. _linux-roadmap:

5. TỐI ƯU LỘ TRÌNH 20 LEVEL LINUX (PHƯƠNG PHÁP JUST-IN-TIME)
==============================================================

Không học tuần tự từ Level 0 đến Level 20 để tránh chán nản và ngợp kiến thức. Áp dụng phương pháp **Just-In-Time (JIT) Learning**, chỉ đào sâu các Level trực tiếp phục vụ cho Performance & AI Systems.

.. list-table:: Định hướng phân bổ 20 Level Linux
   :widths: 30 20 50
   :header-rows: 1

   * - Nhóm Level Linux
     - Ưu tiên
     - Mục tiêu ứng dụng cho Edge AI Systems
   * - **Level 1: System Programming**
     - RẤT CAO
     - Quản lý Thread, POSIX Mutex, Shared Memory, IPC, Signal handling cho Pipeline Audio/AI.
   * - **Level 5 - 8: Kernel Modules, Char Driver, Interrupt, Platform Driver**
     - TRUNG BÌNH
     - Viết/Sửa driver cho cảm biến I2S Mic, Camera, NPU HAL Driver đơn giản.
   * - **Level 9 - 10: Kernel Memory & Synchronization**
     - RẤT CAO
     - Hiểu Page Allocation, DMA Buffers, Zero-copy Memory, Lock-free Queues để truyền dữ liệu cho mô hình AI nhanh nhất.
   * - **Level 11 & 16: Kernel Debug & Performance Tuning**
     - RẤT CAO
     - Sử dụng ``perf``, ``ebpf``, ``ftrace``, ``tracepoints`` để profiling và loại bỏ Latency Spikes khi Inference.
   * - **Level 15: Power Management**
     - RẤT CAO
     - Quản lý DVFS, Thermal Throttling khi mô hình AI bắt đầu ngốn CPU/NPU trên thiết bị pin.
   * - **Level 17 - 19: Buildroot, Yocto, Bootloader**
     - CƠ BẢN
     - Chỉ học vừa đủ để build một bản Linux nhẹ (Minimal Linux OS) cho bo mạch nhúng.

.. _kanban-board:

6. BẢNG QUẢN LÝ TIẾN ĐỘ (KANBAN BOARD)
========================================

.. list-table:: Nhật ký Tiến độ & Mục tiêu
   :widths: 20 50 30
   :header-rows: 1

   * - Hạng mục / Dự án
     - Chủ đề / Công việc
     - Trạng thái
   * - **Project 1**
     - ESP32 Dashboard & RF24 HAL (C Clean, Event-driven)
     - **ĐÃ HOÀN THÀNH**
   * - **C++ Upgrade**
     - Modern C++ (Smart Pointers, RAII, Move Semantics, Concurrency)
     - **ĐANG THỰC HIỆN**
   * - **LeetCode Core**
     - 30-50 bài System-related (Bitwise, Sliding Window, Ring Buffer, Graph DAG)
     - **ĐANG THỰC HIỆN**
   * - **Capstone Project**
     - Edge AI Audio Translate System (Whisper.cpp + ESP32 + Linux Perf)
     - **MỤC TIÊU TRỌNG TÂM**
   * - **AI Optimization**
     - Thực hành Quantization (INT8), Benchmark ONNX Runtime vs TFLite
     - **CHƯA BẮT ĐẦU**