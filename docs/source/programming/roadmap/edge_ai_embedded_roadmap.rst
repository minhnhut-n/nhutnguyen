==================
Lộ trình Edge AI
==================

:Tác giả: Kỹ sư Edge AI
:Ngày tạo: 2026
:Trạng thái: Đang thực hiện
:Định hướng cốt lõi: Tập trung vào Edge AI & AI on Embedded Systems. Tận dụng lợi thế phần cứng, xử lý thời gian thực (real-time), tối ưu năng lượng, viết driver, tối ưu tài nguyên và độ tin cậy.

.. contents:: Mục lục
   :depth: 2
   :local:


1. AI nền tảng vừa đủ
======================

Mục tiêu
  Hiểu bản chất cách mô hình hoạt động và đánh giá được chất lượng mô hình (không tập trung làm researcher).

1.1 Công cụ cốt lõi
---------------------
- [ ] **Ngôn ngữ & Thư viện:** Python, NumPy, Pandas.
- [ ] **Toán ứng dụng:** Trực giác về xác suất và thống kê.

1.2 Machine Learning cơ bản
--------------------------------
- [ ] **Thuật toán chính:**
  - Phân loại (Classification)
  - Hồi quy (Regression)
- [ ] **Kỹ thuật quan trọng:**
  - Hiện tượng quá khớp (Overfitting) & Cách khắc phục
  - Chia tập dữ liệu (Train / Validation / Test)
  - Các chỉ số đánh giá (Accuracy, Precision, Recall, F1-score, MSE...)

.. note::
   *Ghi chú học tập Giai đoạn 1:*

   - [Thêm ghi chú/tài liệu tham khảo tại đây]


2. Deep Learning cho Tín hiệu & Thị giác
==========================================

Mục tiêu
  Nắm vững các kiến trúc mạng cơ bản phục vụ cho xử lý dữ liệu cảm biến, âm thanh và hình ảnh.

2.1 Framework & Mô hình cốt lõi
---------------------------------
- [ ] **Framework:** PyTorch (Lựa chọn ưu tiên để học và thử nghiệm).
- [ ] **Mạng CNN:** Dùng cho xử lý ảnh và thị giác máy tính.
- [ ] **Mạng cho dữ liệu chuỗi:** RNN/LSTM/1D-CNN cho dữ liệu IMU, audio, cảm biến.

2.2 Bài toán thực hành sát với Nhúng
--------------------------------------
- [ ] Nhận dạng cử chỉ từ cảm biến IMU (Gesture Recognition).
- [ ] Phát hiện bất thường của động cơ / độ rung (Anomaly Detection).
- [ ] Nhận diện từ khóa giọng nói (Keyword Spotting - KWS).

.. note::
   *Ghi chú học tập Giai đoạn 2:*

   - [Thêm ghi chú/tài liệu tham khảo tại đây]


3. Chuyển mô hình xuống Thiết bị biên
======================================

Mục tiêu
  Làm chủ quá trình tối ưu và chuyển đổi mô hình từ môi trường Train xuống phần cứng giới hạn tài nguyên.

3.1 Công cụ Chuyển đổi & Tối ưu
---------------------------------
- [ ] **ONNX:** Định dạng mô hình trung gian.
- [ ] **TensorFlow Lite / TFLite Micro:** Runtime cho vi điều khiển và thiết bị biên.
- [ ] **Kỹ thuật Lượng tử hóa:** Quantization (INT8 / Float16).

3.2 Kỹ năng đánh đổi (Trade-off Analysis)
-------------------------------------------
Cân bằng 4 yếu tố cốt lõi của một Kỹ sư Edge AI:

.. table:: Các tiêu chí đánh đổi trong Edge AI
   :widths: 25 75

   ====================== ====================================================
   Tiêu chí               Mô tả & Mục tiêu tối ưu
   ====================== ====================================================
   **Độ chính xác**       Accuracy / F1-Score của mô hình sau khi nén.
   **Độ trễ**             Latency (thời gian Inference/xử lý mỗi khung hình).
   **Bộ nhớ**             Dung lượng RAM / Flash tiêu tốn.
   **Điện năng**          Mức tiêu thụ năng lượng (Power Consumption).
   ====================== ====================================================

.. note::
   *Ghi chú học tập Giai đoạn 3:*

   - [Thêm ghi chú/tài liệu tham khảo tại đây]


4. Thực hành trên Nền tảng Phần cứng
======================================

Mục tiêu
  Triển khai mô hình AI lên các nền tảng chip thực tế từ Vi điều khiển (MCU) đến SoC Linux.

4.1 Dòng Vi điều khiển (MCU / TinyML)
---------------------------------------
- [ ] **Hardware:** STM32, ESP32, nRF, Renesas, NXP...
- [ ] **Software/Framework:** TinyML, TFLite Micro, CMSIS-NN.

4.2 Hệ điều hành Nhúng (Embedded Linux)
-----------------------------------------
- [ ] **Hardware:** Raspberry Pi, NVIDIA Jetson, i.MX, Rockchip...
- [ ] **Inference Engine:** ONNX Runtime, TensorRT, OpenVINO.

4.3 Nâng cao (Phần cứng chuyên dụng)
--------------------------------------
- [ ] **FPGA / NPU:** Nghiên cứu khi công việc yêu cầu hiệu năng cực cao hoặc xử lý luồng Camera song song.

.. note::
   *Ghi chú học tập Giai đoạn 4:*

   - [Thêm ghi chú/tài liệu tham khảo tại đây]


5. MLOps cho Thiết bị thật
============================

Mục tiêu
  Đưa mô hình vào hệ thống vận hành thực tế một cách an toàn, ổn định và có thể bảo trì.

5.1 Kiến thức Quản lý & Vận hành
---------------------------------
- [ ] Quản lý phiên bản dữ liệu & mô hình (Dataset / Model Versioning).
- [ ] Do kiểm hiệu năng thực tế (Benchmarking & Logging).
- [ ] Quy trình cập nhật & khôi phục mô hình (Model Update & Rollback).

5.2 Thách thức đặc thù trên Thiết bị Biên
-------------------------------------------
- [ ] **Data Drift:** Xử lý hiện tượng trôi dữ liệu khi môi trường thực tế thay đổi.
- [ ] **Secure OTA:** Cập nhật mô hình qua mạng an toàn, chống lỗi gạch thiết bị (bricking).
- [ ] **Privacy:** Đảm bảo quyền riêng tư dữ liệu ngay tại thiết bị biên.
- [ ] **Reproducibility:** Khả năng tái lập lỗi và kết quả kiểm thử.

.. note::
   *Ghi chú học tập Giai đoạn 5:*

   - [Thêm ghi chú/tài liệu tham khảo tại đây]


Bảng theo dõi tiến độ học tập
=================================

.. list-table:: Nhật ký Học tập & Thực hành
   :widths: 15 20 45 20
   :header-rows: 1

   * - Giai đoạn
     - Chủ đề
     - Dự án / Bài thực hành
     - Trạng thái
   * - Giai đoạn 1
     - ML Basics
     - Phân loại dữ liệu cảm biến đơn giản
     - Chưa bắt đầu
   * - Giai đoạn 2
     - PyTorch & CNN
     - Nhận diện Keyword Spotting (KWS)
     - Chưa bắt đầu
   * - Giai đoạn 3
     - Quantization
     - Nén mô hình về INT8 bằng TFLite
     - Chưa bắt đầu
   * - Giai đoạn 4
     - Embedded AI
     - Chạy mô hình KWS trên ESP32/STM32
     - Chưa bắt đầu
   * - Giai đoạn 5
     - Edge MLOps
     - Lập quy trình cập nhật mô hình qua OTA
     - Chưa bắt đầu