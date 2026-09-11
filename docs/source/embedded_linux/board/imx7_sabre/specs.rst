===============================================================================
1. Board Specifications — i.MX 7Dual SabreSD
===============================================================================

.. meta::
   :description: Technical specifications of the NXP i.MX 7Dual SabreSD development board
   :keywords: i.MX7, SabreSD, SABRE-SD, NXP, Embedded Linux, ARM Cortex-A7, Cortex-M4

.. contents:: **Mục lục chi tiết**
   :depth: 2
   :local:
   :backlinks: entry

---

Tổng quan
=========

Bo mạch **NXP i.MX 7Dual SabreSD (SABRE-SD REV C4)** là development board chính
thức của NXP dựa trên SoC **i.MX 7Dual (MCIMX7D)**. Điểm nổi bật của kiến trúc
này là mô hình **heterogeneous dual-core**: một cặp Cortex-A7 chạy Linux và một
Cortex-M4 chạy firmware RTOS/bare-metal, chia sẻ tài nguyên ngoại vi qua
**RDC (Resource Domain Controller)**.

* **Official docs:** `i.MX 7Dual Documentation (nxp.com) <https://www.nxp.com/products/i-mx-7dual-applications-processors-integrating-dual-arm-cortex-a7-cores-and-cortex-m4-core:i.MX7D>`_
* **Evaluation kit:** MCIMX7D-SDB (SABRE Development Board)

---

1. SoC & Kiến trúc
==================

.. list-table:: **Thông số chính của SoC i.MX 7Dual**
   :widths: 30 70
   :header-rows: 1

   * - Thành phần
     - Chi tiết
   * - **CPU (Application)**
     - 2× **ARM Cortex-A7** @ 1.0 GHz (32-bit, ARMv7-A), có NEON, VFPv4, L2 cache 1MB
   * - **CPU (Real-time)**
     - 1× **ARM Cortex-M4** @ 200 MHz — chạy FreeRTOS/firmware riêng, độc lập với Linux
   * - **GPU**
     - Vivante GC308 (2D) + GC2010 (2D vector) — không có 3D GPU
   * - **Memory interface**
     - 32-bit LPDDR3 / DDR3 / DDR3L / LPDDR2
   * - **Boot ROM**
     - ROM 96KB hỗ trợ boot từ SD/eMMC, QSPI, NAND, USB Serial Download
   * - **Heterogeneous**
     - RDC + MU (Messaging Unit) + CCN: chia sẻ DDR, peripherals giữa A7 và M4

.. note::
   Khác với i.MX 6/8, i.MX 7 **không có** GPU 3D và encoding/decoding video
   bằng phần cứng (VPU) — SoC này hướng tới low-power, IoT, HMI đơn giản và
   industrial control.

---

2. Bộ nhớ & Lưu trữ trên SabreSD
================================

.. list-table:: **Cấu hình bộ nhớ mặc định trên board**
   :widths: 35 40 25
   :header-rows: 1

   * - Loại
     - Dung lượng / Chuẩn
     - Vị trí sử dụng
   * - LPDDR3
     - 1GB (32-bit @ 400MHz / 800MT/s)
     - RAM chạy Linux + code cho M4 (theo region)
   * - eMMC
     - 4GB hoặc 8GB (SDIO4)
     - Boot media chính (USDHC3)
   * - SD Card
     - Slot microSD (USDHC1)
     - Boot media thứ hai
   * - QSPI NOR
     - 16MB @ 133MHz (Quad SPI)
     - Chứa M4 image, SCU config, backup bootloader

---

3. Giao tiếp & Ngoại vi
=======================

USB
---
* **USB OTG (HS)** — dùng cho **Serial Download Mode** (UUU/mfgtools, SDP)
* **USB Host** qua OTG adapter

UART / Console
--------------
* **UART1 (USDHC1 connector, J24 — Debug port)** — console mặc định
  ``115200 8N1``, dùng để debug bootloader/kernel.
* UART2, UART5, UART6, UART7 dẫn ra header.

Networking
----------
* **Ethernet:** 1× 10/100 PHY (Micrel KSZ8041) qua RJ45

Display & Multimedia
---------------------
* **MIPI-DSI** 4 lanes
* **LCDIF** — parallel 24-bit RGB LCD
* **MIPI-CSI** camera input
* **EPDC** — E-Ink display controller (đặc trưng của i.MX 7)

Audio
-----
* SAI/I2S, S/PDIF, codec WM8960 (mặc định trên SabreSD)

Expansion
---------
* **Arduino UNO compatible header** (cổng R3)
* Header GPIO, I2C, SPI (ECSPI1–4), CAN (FlexCAN1/2), PWM
* **JTAG header** — debug qua JTAG/Lauterbach/OpenOCD

---

4. Nguồn & Vận hành
===================

* Nguồn cấp: **5V DC** qua barrel jack hoặc USB OTG
* Tiêu thụ điện năng rất thấp (thiết kế cho low-power IoT — có hỗ trợ
  các chế độ **Suspend-to-RAM / DSM**, deep sleep từ vài mW)
* Hỗ trợ **PMIC PF3000** quản lý nguồn, thứ tự power-up

---

5. Phần mềm hỗ trợ
==================

.. list-table:: **Phần mềm reference của NXP cho i.MX 7Dual SabreSD**
   :widths: 30 70
   :header-rows: 1

   * - Thành phần
     - Nguồn chính thức
   * - **BSP / Yocto**
     - `i.MX Yocto Project BSP (imx-manifest) <https://github.com/nxp-imx/imx-manifest>`_ — meta-imx, imx-base
   * - **U-Boot**
     - `u-boot-imx <https://github.com/nxp-imx/uboot-imx>`_ — ``mx7dsabresd_defconfig``
   * - **Kernel**
     - `linux-imx <https://github.com/nxp-imx/linux-imx>`_ — defconfig ``imx_v7_defconfig``
   * - **Device Tree**
     - ``arch/arm/boot/dts/imx7d-sdb.dts`` (NXP) hoặc ``imx7d-sdb-revc.dts``
   * - **Firmware M4**
     - `imx-mkimage + multicore examples (FreeRTOS) <https://github.com/nxp-mcuxpresso>`_ — build bằng **MCUXpresso SDK** với board ``evkmcimx7d``
   * - **Burning tool**
     - `UUU (Universal Update Utility) <https://github.com/nxp-imx/mfgtools>`_ — thay thế mfgtools cũ

.. tip::
   Khi build Yocto cho board này: ``MACHINE=imx7dlsabresd`` (hoặc
   ``imx7dsabresd`` tùy bản BSP) — dùng ``fsl-image-validation-imx``.

---

.. important::
   Tất cả thông tin trên là cấu hình **mặc định của SabreSD REV-C4**.
   Khi tự design carrier board hoặc dùng board khác trong cùng family
   (i.MX7 Solo SABRE), cần kiểm tra lại schematic tương ứng.