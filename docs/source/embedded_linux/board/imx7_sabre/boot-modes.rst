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

Board **i.MX 7Dual SabreSD** tích hợp nhiều **boot mode**, mỗi chế độ nạp
boot image từ một nguồn lưu trữ khác nhau. Việc chọn boot mode được cấu hình
hoàn toàn bằng **phần cứng**, thông qua các công tắc DIP trên board:

* ``SW1`` — công tắc chọn **nguồn boot** (boot source).
* ``SW2``, ``SW3`` — hai công tắc chọn **boot mode**.

.. note::
   Ký hiệu bit (``0``/``1``/``X``) được in **ngay cạnh từng công tắc** trên
   bề mặt board (silk-screen). Hãy đối chiếu trực tiếp các ký hiệu này khi
   gạt công tắc.

Bảng dưới đây tổng hợp vị trí công tắc cho từng boot media — trích từ tài
liệu chính thức của NXP:

.. list-table:: **Table 1. Booting from SD1/J6 on i.MX7Dual Sabre-SD**
   :widths: 30 40 30
   :header-rows: 1
   :align: center

   * - Boot Media
     - ``SW2`` [D1-D8]
     - ``SW3`` [D1-D2]
   * - SD card (``SD1``)
     - ``00100000``
     - ``10``
   * - eMMC
     - ``01010000``
     - ``10``
   * - NAND
     - ``011XXXX0``
     - ``10``
   * - QuadSPI
     - ``10000000``
     - ``10``
   * - SDP (Serial Download)
     - ``XXXXXXXX``
     - ``01``

.. tip::
   Nguồn tham khảo chính thức: `Getting Started with the MCIMX7SABRE (NXP)
   <https://www.nxp.com/document/guide/getting-started-with-the-mcimx7sabre:GS-MCIMX7SABRE?section=out-of-the-box&subSection=out-of-the-box-5>`_

---

Chế độ 1: Boot từ SD Card
=========================

**Mục đích:** chạy **U-Boot + Linux** từ thẻ microSD — chế độ phổ biến nhất
trong giai đoạn **development**.

Chuẩn bị
--------

.. list-table:: **Danh mục chuẩn bị**
   :widths: 22 28 50
   :header-rows: 1

   * - Hạng mục
     - Yêu cầu
     - Chi tiết
   * - **Thẻ microSD**
     - Tối thiểu **8 GB**
     - Ghi sẵn **image Yocto** hoặc **image nhà sản xuất cung cấp** (thường
       là kernel ``4.14``, image chỉ chiếm **< 1 GB**)
   * - **Console UART**
     - ``115200 8N1``
     - Cắm vào **cổng DEBUG UART** trên board. Trên PC là ``COM14`` (hoặc
       cổng COM tương ứng — nhận diện qua **LED chỉ thị** sáng cạnh dòng
       chữ *UART debug* khi cắm cáp)
   * - **Nguồn cấp**
     - **5V DC, 2-4 A**
     - Dòng đủ lớn để boot chắc chắn; nếu muốn dư dả headroom thì dùng luôn
       **5V/5 A** (adapter đi kèm kit NXP)

Các bước thực hiện
------------------

#. **Ghi image vào thẻ microSD** — flash image Yocto hoặc image NXP vào
   thẻ.

   .. code-block:: bash

      # Ví dụ trên Linux — thay sdX bằng device của thẻ SD
      $ dd if=<image>.wic of=/dev/sdX bs=1M status=progress conv=fsync

   Trên Windows có thể dùng *balenaEtcher* hoặc *Win32DiskImager*.

#. **Cấu hình công tắc boot** theo **Table 1**:

   .. code-block:: text

      SD card (SD1):  SW2 = 00100000 | SW3 = 10

#. **Gắn thẻ microSD** vào slot **SD1 (J6)**.

#. **Kết nối UART debug** với PC, mở terminal (*PuTTY* / *Tera Term*) với
   cấu hình ``115200 8N1`` trên cổng COM đã xác định ở phần chuẩn bị.

#. **Cấp nguồn 5V** cho board.

#. **Theo dõi log boot** trên terminal — chuỗi log đi qua
   *BootROM → U-Boot → kernel*. Log in ra liên tục nghĩa là board đã boot
   thành công từ SD.

Lưu ý
-----

.. note::
   Vị trí công tắc boot chỉ được BootROM đọc **tại thời điểm reset /
   power-on**. Nếu đổi công tắc khi board đang chạy, cần power-cycle lại
   để cấu hình mới có hiệu lực.

.. warning::
   Không tháo/phục thẻ microSD khi board đang cấp nguồn hoặc hệ thống đang
   chạy *(thực tế không nghiêm trọng — chỉ là quy tắc nên tuân theo)*.

   Trong quá trình vận hành có thể gặp trường hợp sau:
   
   * **Không boot được, log báo lỗi** ``1.8V power fail`` — nguồn cấp cho
     thẻ SD không đủ. **Giải pháp:** dùng **adapter 5V/5A** hoặc **thẻ SD
     khác**.

     *Trường hợp thực tế:* lỗi xuất hiện khi nguồn chỉ **5V/3A** và thẻ SD
     đời cũ (board để lâu không sử dụng, linh kiện có hiện tượng oxy hóa).
     Dùng tạm **thẻ SD đời mới hơn** để boot mồi lần đầu — các lần boot
     sau có thể dùng lại thẻ cũ bình thường.

---

Chế độ 2: Boot từ eMMC
======================

**Mục đích:** chạy **U-Boot + Linux** từ bộ nhớ **eMMC onboard** — không phụ
thuộc thẻ SD rời.

Chuẩn bị
--------

.. note::
   **TODO:** bổ sung danh mục chuẩn bị (image, công cụ ghi, console,
   nguồn).

Vị trí công tắc boot theo **Table 1**:

.. code-block:: text

   eMMC:  SW2 = 01010000 | SW3 = 10

Các bước thực hiện
------------------

.. note::
   **TODO:** bổ sung các bước chi tiết.

Lưu ý
-----

.. note::
   **TODO:** bổ sung lưu ý khi vận hành.

---

Chế độ 3: Boot từ NAND
======================

**Mục đích:** chạy **U-Boot + Linux** từ flash **NAND**.

Chuẩn bị
--------

.. note::
   **TODO:** bổ sung danh mục chuẩn bị (flash module, image, công cụ
   ghi/erase, console, nguồn).

Vị trí công tắc boot theo **Table 1**:

.. code-block:: text

   NAND:  SW2 = 011XXXX0 | SW3 = 10

Các bước thực hiện
------------------

.. note::
   **TODO:** bổ sung các bước chi tiết.

Lưu ý
-----

.. note::
   **TODO:** bổ sung lưu ý khi vận hành.

---

Chế độ 4: Boot từ QSPI NOR
==========================

**Mục đích:** nạp boot image từ flash **QSPI NOR onboard** (16 MB).

Chuẩn bị
--------

.. note::
   **TODO:** bổ sung danh mục chuẩn bị (image cho QSPI, công cụ ghi,
   console, nguồn).

Vị trí công tắc boot theo **Table 1**:

.. code-block:: text

   QuadSPI:  SW2 = 10000000 | SW3 = 10

Các bước thực hiện
------------------

.. note::
   **TODO:** bổ sung các bước chi tiết.

Lưu ý
-----

.. note::
   **TODO:** bổ sung lưu ý khi vận hành.

---

Chế độ 5: Serial Download (SDP)
===============================

**Mục đích:** nạp image vào board **qua USB** từ máy host (UUU / mfgtools)
— dùng để **flash boot media** hoặc **recover** board khi không boot được.

Chuẩn bị
--------

.. note::
   **TODO:** bổ sung danh mục chuẩn bị (UUU, cáp USB OTG, image,
   console).

Vị trí công tắc boot theo **Table 1**:

.. code-block:: text

   SDP:  SW2 = XXXXXXXX | SW3 = 01

Các bước thực hiện
------------------

.. note::
   **TODO:** bổ sung các bước chi tiết.

Lưu ý
-----

.. note::
   **TODO:** bổ sung lưu ý khi vận hành.

---

.. important::
   Nội dung trong tài liệu này đã được **xác minh trên board thực tế** và
   đối chiếu với tài liệu chính thức của NXP.

