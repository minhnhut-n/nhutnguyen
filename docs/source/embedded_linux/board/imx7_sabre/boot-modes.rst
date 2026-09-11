===============================================================================
2. User Guide — Boot with Modes (i.MX 7Dual SabreSD)
===============================================================================

.. meta::
   :description: Hướng dẫn boot i.MX 7Dual SabreSD qua các chế độ: Serial Download, SD/eMMC, QSPI
   :keywords: i.MX7, SabreSD, boot mode, serial download, UUU, u-boot, SDP, USB

.. contents:: **Mục lục chi tiết**
   :depth: 2
   :local:
   :backlinks: entry

---

Tổng quan các chế độ Boot
=========================

i.MX 7Dual có **Boot ROM** tích hợp sẵn. Khi board reset, ROM sẽ đọc trạng
thái của **Boot Mode Switches (SW6)** và eFuse để quyết định phương thức
boot. Trên SabreSD, **SW6** là công tắc DIP 4-bit.

.. list-table:: **Cấu hình SW6 (Boot Mode Switch) trên SabreSD**
   :widths: 20 30 50
   :header-rows: 1

   * - SW6[4:1]
     - Chế độ boot
     - Mô tả
   * - ``0000``
     - **FlexSPI / QSPI NOR**
     - Boot từ QSPI NOR Flash 16MB (chứa M4 firmware/backup bootloader)
   * - ``0010``
     - **Serial Download (SDP)**
     - ROM chờ lệnh từ USB OTG — dùng để **flash image** qua UUU/mfgtools
   * - ``0011``
     - **eMMC (USDHC3)**
     - Boot từ eMMC nội bộ
   * - ``0100``
     - **SD Card (USDHC1)**
     - Boot từ thẻ microSD (slot phía dưới board)

.. note::
   Giá trị bit đọc theo thứ tự **SW6-4 SW6-3 SW6-2 SW6-1** (SW6-1 là bit
   thấp nhất). Cần đối chiếu lại schematic SabreSD của bạn trước khi
   thực hiện — bản REV C4 có thể khác chút về thứ tự.

---

Chế độ 1: Boot từ SD Card
=========================

**Mục đích:** Chạy U-Boot + Linux từ thẻ microSD (dùng cho development).

Các bước:
---------
1. **Chuẩn bị SD card** — ghi bootloader ra đúng offset của ROM:

   .. code-block:: bash

      # u-boot-imx build với mx7dsabresd_defconfig
      sudo dd if=u-boot-dtb.imx of=/dev/sdX bs=1k seek=1 conv=fsync

   .. warning::
      i.MX7 ROM đọc bootloader từ offset **1KB** của media. Không dùng
      ``seek=0`` như các SoC khác.

2. Tạo phân vùng và ghi rootfs (rootfs.ext4 hoặc rootfs.tar giải nén).

3. **Cài SW6 về chế độ SD:**

   .. code-block:: text

      SW6 = [OFF ON OFF OFF]   # 0100 — boot từ USDHC1 (SD)

4. Kết nối **serial console** vào debug UART (J24), terminal
   ``115200 8N1`` rồi reset board.

5. Quan sát output U-Boot:

   .. code-block:: text

      U-Boot 2020.04-imx_v2020.04
      CPU:   Freescale i.MX7D rev1.2
      Board: i.MX7D SABRESD
      Boot:  SD1
      Hit any key to stop autoboot:  3

---

Chế độ 2: Boot từ eMMC
======================

**Mục đích:** Boot từ bộ nhớ trong eMMC — production, không cần SD card.

1. **Boot tạm từ SD hoặc USB** để có môi trường ghi eMMC, hoặc flash eMMC
   từ PC qua Serial Download (xem Chế độ 3).

2. Từ U-Boot, ghi bootloader vào eMMC (device ``mmc 1`` tương ứng eMMC):

   .. code-block:: text

      => mmc dev 1
      => tftp 0x80800000 u-boot-dtb.imx
      => mmc write 0x80800000 0x2 0x400   # offset 1KB = sector 2

3. Cài SW6:

   .. code-block:: text

      SW6 = [OFF OFF ON ON]   # 0011 — boot từ eMMC (USDHC3)

4. Reset board → U-Boot sẽ load từ eMMC.

.. tip::
   Trong U-Boot dùng lệnh ``mmc dev`` và ``mmc info`` để xác định device
   nào là SD, device nào là eMMC trên board của bạn.

---

Chế độ 3: Serial Download Mode (USB SDP) — Flash qua UUU
=========================================================

**Mục đích:** Khi board **chưa có bootloader** hoặc bạn muốn nạp image mới
từ PC qua USB OTG. Đây là chế độ quan trọng nhất khi **recover board**.

**Cách hoạt động:** ROM của i.MX7 hỗ trợ **SDP (Serial Download Protocol)**
trên USB OTG. ROM chờ PC gửi lệnh nạp code vào RAM qua công cụ **UUU**.

1. **Cài SW6 về Serial Download** và cắm USB OTG (J301) vào PC:

   .. code-block:: text

      SW6 = [OFF OFF ON OFF]   # 0010 — Serial Download

2. **Cài UUU trên PC:**

   .. code-block:: bash

      # Linux
      git clone https://github.com/nxp-imx/mfgtools && cd mfgtools
      mkdir build && cd build && cmake .. && make

      # Windows: download uuu.exe từ release page

3. **Flash image qua UUU:**

   .. code-block:: bash

      # Flash bootloader vào eMMC
      sudo ./uuu -b emmc u-boot-dtb.imx

      # Flash full image (bootloader + kernel + rootfs)
      sudo ./uuu -b emmc_all imx-image-core-imx7dlsabresd.wic

      # Flash vào SD card thay vì eMMC
      sudo ./uuu -b sd_all imx-image-core-imx7dlsabresd.wic

4. Sau khi flash xong, đổi SW6 về chế độ boot SD/eMMC và reset.

.. note::
   Nếu UUU không nhận board, kiểm tra:

   - USB OTG cắm đúng port (OTG, không phải USB Host)
   - ``lsusb`` thấy device **NXP Semiconductors (15a2)** với ID ``0076``
   - Quyền truy cập USB (udev rule hoặc chạy ``sudo``)

---

Chế độ 4: Boot từ QSPI NOR
==========================

**Mục đích:** Boot từ QSPI Flash 16MB — chứa firmware cho Cortex-M4 hoặc
backup bootloader.

1. Ghi firmware vào QSPI từ U-Boot:

   .. code-block:: text

      => sf probe
      => tftp 0x80800000 m4_image.bin
      => sf erase 0x0 0x100000
      => sf write 0x80800000 0x0 ${filesize}

2. Cài SW6:

   .. code-block:: text

      SW6 = [OFF OFF OFF OFF]   # 0000 — Boot từ QSPI NOR

3. Reset → ROM đọc QSPI tại offset 0.

.. warning::
   QSPI Boot cần boot ROM nhận diện đúng **format header** của image
   (IVT + DCD). Nếu image M4 không đúng format, ROM sẽ báo lỗi
   ``Bad Magic Number`` qua console.

---

Thứ tự Boot Fallback
====================

Nếu chế độ boot được chọn **fail** (image corrupt, không đọc được), Boot
ROM sẽ fallback:

.. code-block:: text

   1. Boot media được chọn (SD / eMMC / QSPI)
   2. Nếu fail → USB Serial Download (SDP)   ← luôn là fallback cuối

Nhờ vậy, **board không bao giờ "brick" hoàn toàn** — luôn có thể đưa về
Serial Download để nạp lại image qua UUU.

---

Troubleshooting
===============

.. list-table:: **Lỗi thường gặp khi boot**
   :widths: 35 65
   :header-rows: 1

   * - Hiện tượng
     - Nguyên nhân & cách xử lý
   * - Không thấy output UART console
     - Sai baudrate (phải là **115200**), sai UART (debug UART J24), sai chiều TX/RX
   * - UUU không detect board
     - SW6 chưa về Serial Download mode, thiếu driver/udev permission USB
   * - ``Bad Magic Number`` / CRC error
     - Bootloader ghi sai offset (phải là **1KB offset** cho i.MX7), image corrupt
   * - Kernel panic ở rootfs
     - Sai ``root=`` cmdline, sai partition layout, rootfs chưa được flash
   * - ``mmc write`` không có tác dụng
     - Chưa chọn đúng device — dùng ``mmc dev <n>`` và ``mmc info`` để kiểm tra

---

**Tham khảo:**

- `i.MX 7Dual Applications Processor Reference Manual (IMX7DRM)` — chương Boot
- `SABRE-SD Schematic (SCH-27114)` — bản REV C4
- `UUU Documentation <https://github.com/nxp-imx/mfgtools>`_
- `i.MX Yocto Project User's Guide (IMXLXYOCTOUG)` — chương Boot and Update the Board
