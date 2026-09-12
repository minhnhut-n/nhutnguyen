===================================================================
1. Linux Kernel Subsystems Guide for AI Systems & Infrastructure
===================================================================

:Author: Embedded & AI Infrastructure Engineer
:Target Domain: High-Performance AI Inference, Kernel Tuning, Low-Latency Execution
:Kernel Versions: 4.9 LTS (Legacy CFS) & 6.12+ (Modern EEVDF)

Overview
========
Tài liệu tổng hợp các Kernel Subsystems và Modules quan trọng giúp tối ưu hóa tài nguyên phần cứng (CPU, RAM, DMA, I/O) cho các ứng dụng và dịch vụ AI/Inference hiệu năng cao.

.. contents:: Table of Contents
   :depth: 2
   :local:

1. Scheduler & Process Management
=================================
Đảm bảo CPU được ưu tiên cho tác vụ AI, giảm trễ (Latency) và hạn chế giật lag (Jitter).

* **kernel/sched (CFS, EEVDF, Real-Time Scheduler)**
  * Hiểu cơ chế tính toán Virtual Runtime (``vruntime``) trên CFS (Kernel 4.9) và Virtual Deadline / Latency Slices trên EEVDF (Kernel 6.12+).
  * Phân biệt các chính sách lập lịch: ``SCHED_OTHER``, ``SCHED_FIFO``, ``SCHED_RR``, và ``SCHED_DEADLINE``.
  * Làm chủ các System Calls: ``sched_setscheduler()``, ``sched_setattr()``, và ``sched_setaffinity()``.

* **cpufreq / cpuidle (Power & Governor Management)**
  * Hiểu cơ chế hoạt động của các Governors: ``performance``, ``powersave``, ``schedutil``.
  * Tương tác với sysfs (``/sys/devices/system/cpu/cpu*/cpufreq/``) để nâng xung nhịp CPU lên tối đa ngay khi kích hoạt AI Task.

* **cgroups v2 (cpu, cpuset, memory)**
  * Quản lý giới hạn tài nguyên cứng: ``cpu.max``, ``cpu.weight``, ``memory.max``, ``memory.high``.
  * Cô lập CPU (``isolcpus``, ``nohz_full``) để dành riêng Core vật lý cho AI Inference.

2. Memory Management & Zero-Copy Subsystem
==========================================
Tối ưu hóa băng thông bộ nhớ, triệt tiêu chi phí copy dữ liệu giữa Kernel Space và User Space.

* **mm (Virtual Memory, Page Allocator, Paging & Swap)**
  * Nắm vững System Calls: ``mmap()``, ``madvise()``, và **``mlock()`` / ``mlockall()``** (khóa Weights/Tensors trong RAM vật lý, chống Paging/Swap ra ổ đĩa).

* **dma-buf (DMA Buffer Sharing Framework)**
  * **Module cốt lõi:** Chia sẻ con trỏ bộ nhớ (Buffer FD) trực tiếp giữa Camera/Sensor Driver, NPU/GPU Driver và AI Runtime mà **không qua memcpy (Zero-Copy)**.

* **CMA (Contiguous Memory Allocator) / DMABUF-HEAPS**
  * Định nghĩa và cấp phát vùng nhớ vật lý liên tục dung lượng lớn trong Device Tree (``.dts``) dành riêng cho Tensor allocations.

* **HUGETLBFS / Transparent Huge Pages (THP)**
  * Tận dụng Huge Pages (2MB / 1GB) để giảm tỷ lệ trễ Cache Miss (TLB Miss) khi load Model Weights dung lượng lớn.

3. Hardware Interface & Inter-Process Communication
===================================================
Nạp dữ liệu từ thiết bị ngoại vi vào Pipeline tính toán AI.

* **v4l2 (Video for Linux 2) & Media Subsystem**
  * Khai thác luồng dữ liệu từ Camera vào khung RAM chung bằng ``V4L2_MEMORY_DMABUF``.

* **char/mem, UIO (Userspace I/O) & VFIO**
  * Điều khiển thanh ghi phần cứng phần vùng (GPU/NPU/FPGA) trực tiếp từ User-space để đạt hiệu năng tối đa.

* **net/core & Socket (eBPF / XDP)**
  * Xử lý luồng dữ liệu mạng (Network Packet Ingestion) cấp độ Kernel cho các AI Inference Servers.

4. Profiling, Tracing & Performance Metrics
===========================================
Đo đạc chính xác độ trễ và điểm nghẽn hiệu năng của hệ thống.

* **ftrace & tracepoints**
  * Theo dõi luồng chuyển giao tiến trình (``sched_switch``), đo độ trễ ngắt (Interrupt Latency) và thời gian thực thi của Kernel Functions.

* **perf_events Subsystem**
  * Đếm các sự kiện phần cứng (Hardware Counters): CPU Cache Misses, Branch Mispredictions, Bus Cycles.

* **eBPF (Extended Berkeley Packet Filter)**
  * Gán các Probe (``kprobe``, ``uprobe``) giám sát hành vi và Latency của AI Service theo thời gian thực.

Checklist Thực Hành Từng Bước
============================

1. **Giai đoạn 1 (Scheduler):** Thử nghiệm ``sched_setattr()``, ``sched_setaffinity()`` và ``mlockall()`` trên C/C++ App.
2. **Giai đoạn 2 (Memory & DMA):** Lập trình truyền mảng dữ liệu qua ``dma-buf`` giữa 2 tiến trình mà không dùng ``memcpy``.
3. **Giai đoạn 3 (Profiling):** Dùng ``perf`` và ``ftrace`` trích xuất biểu đồ Latency của AI Task dưới các cấu hình Kernel khác nhau.