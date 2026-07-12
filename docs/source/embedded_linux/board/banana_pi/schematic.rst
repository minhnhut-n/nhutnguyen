Schematics
=============================================================================

.. rubric:: 1. Quy Ước Đặt Tên & Ký Hiệu Cơ Bản

**Ký hiệu pha/cực trong truyền tín hiệu vi sai (Differential Pair):**

- **P / Positive / Plus / +** : Cực dương (Ví dụ: ``USB_DP``, ``CKP``, ``PETp0``).
- **N / Negative / M / Minus / -** : Cực âm (Ví dụ: ``USB_DN``, ``USB_DM``, ``CKM``, ``PETn0``).

**Ký hiệu Logic Âm (Active Low - Kích hoạt ở mức LOW 0V):**

Thường có tiền tố/hậu tố: ``_``, ``N``, ``#``, hoặc ``n``.
Ví dụ: ``RESET_N``, ``_EN``, ``CS#`` (Chip Select Active Low).

**Chữ G (General / Generic):**

Đại diện cho tính linh hoạt (flexible) của ngoại vi, có thể kết nối đa năng (cảm biến, màn hình OLED, RFID...).
Ví dụ: ``GPIO`` (General Purpose Input/Output), ``GSPI`` (General SPI).

---

.. rubric:: 2. Quản Lý Nguồn Điện (Power Rails & Ground)

**Các đường nguồn chính (Power Rails):**

- **VDD** (Voltage Drain Drain): Nguồn cấp cho các linh kiện số (Digital).
  - ``VDD_CPU`` / ``VDD_CORE``: Nguồn cấp riêng cho nhân CPU (thường từ 0.9V - 1.1V).
  - ``VDD_SYS``: Nguồn chính của toàn hệ thống.
  - ``VDD_DRAM``: Nguồn cấp cho bộ nhớ RAM (DDR3/DDR4).
- **VCC** (Collector Collector): Nguồn cấp điện áp dương chính.
  - ``VCC_IO`` / ``VCC_3V3``: Nguồn cấp cho các chân giao tiếp ngoại vi (thường 3.3V).
- **VBUS:** Nguồn 5V cấp trực tiếp từ cổng USB vào.
- **VBAT:** Nguồn cấp từ Pin (Pin RTC 3V hoặc Pin sạc).

**Đường Nối Đất (Ground / Mass):**

- **VSS / GND:** Đường nối đất cực âm (0V).
- **PGND** (Power Ground): Đất công suất (dành cho bộ nguồn xung, cuộn cảm), giúp cách ly nhiễu dòng lớn. Ví dụ: ``PGND1``.
- **AGND** (Analog Ground): Đất âm thanh/hình ảnh analog, giúp khử nhiễu xì/sọc.

**Bản chất Drain & Source trong MOSFET:**

- **Source (S):** Nơi dòng điện đi vào.
- **Drain (D):** Nơi dòng điện đi ra.
- **Gate (G):** Cổng điều khiển van đóng/mở dòng điện.

* Nguyên tắc lưu thông dòng điện:

  * **Đối với Mạch ngoài (External Circuit):**
    * **Dòng điện quy ước:** Đi từ nơi có điện áp cao về điện áp thấp (từ cực Dương **+** sang cực Âm **-**).
    * **Dòng electron thực tế:** Chuyển động ngược lại, từ cực Âm **-** sang cực Dương **+**.

  * **Đối với Mạch trong / Bên trong nguồn điện (Internal Circuit):**
    * **Dòng điện quy ước:** Đi từ cực Âm **-** sang cực Dương **+** để tạo thành một **vòng tuần hoàn kín**.
    * **Cơ chế:** Bên trong nguồn (pin/ắc quy), lực hóa học (lực lạ) thực hiện công để đẩy các điện tích ngược chiều điện trường, giúp duy trì hiệu điện thế. Khi phản ứng hóa học kết thúc (năng lượng giảm về 0) $\rightarrow$ Pin hết điện.
---

.. rubric:: 3. Xung Nhịp & Tín Hiệu Điều Khiển (Clock & Control)

- **RST / RESET / RESET_N:** Tín hiệu khởi động lại (Reboot) hệ thống hoặc thiết bị ngoại vi.
- **EN / _EN (Enable):** Bật/tắt chức năng hoặc khối nguồn tương ứng.
- **CLK / SCLK / Clock:** Đường xung nhịp đồng bộ truyền dữ liệu.

**Chân INSTALL:**

Là chân **GPIO đầu vào (Input)** kéo về chip chính để đọc trạng thái phần cứng.

- *Ứng dụng 1 (Phân biệt phiên bản Board):* Nối 3.3V hoặc GND để U-Boot nhận biết phiên bản board (vd: Option có/không Wi-Fi) và nạp Device Tree tương ứng.
- *Ứng dụng 2 (Nhận biết Cắm/Rút Card):* Khi cắm card M.2 / PCIe / Camera, chân này bị kéo xuống GND → Chip nhận biết thiết bị đã được cắm.

---

.. rubric:: 4. Các Chuẩn Giao Tiếp Dữ Liệu (Data Interfaces)

**Cổng USB & Data Lines:**

- **D (Data):** Đường dữ liệu nối tiếp.
- **D3P / USB_DP / USB_D+:** Đường dữ liệu cực dương (Cổng USB số 3).
- **D3M / USB_DM / USB_D-:** Đường dữ liệu cực âm (Cổng USB số 3).

Số 1, 2, 3 đại diện cho cổng USB trên board.

**Giao tiếp Nối tiếp (Serial & Buses):**

- **UART:** ``TX`` (Transmit - Gửi) / ``RX`` (Receive - Nhận).
- **I2C:** ``SDA`` (Serial Data) / ``SCL`` (Serial Clock).
- **SPI (Serial Peripheral Interface):**
  - ``MOSI`` (Master Out Slave In): Master gửi dữ liệu.
  - ``MISO`` (Master In Slave Out): Master nhận dữ liệu.
  - Phân loại các khối SPI:

=========  ==========  ============================================
Loại SPI   Tên gọi     Chức năng & Hạn chế
=========  ==========  ============================================
General    GSPI        Đa năng: Kết nối được mọi loại ngoại vi
Flash      QSPI/       Đặc chủng: Đọc/ghi chip nhớ Flash (BIOS)
            SPI-NOR
Display    DBI/        Đặc chủng: Truyền dữ liệu hình ảnh ra LCD
            LCD-SPI
=========  ==========  ============================================

**PCIe (PCI Express):**

Chuẩn kết nối tốc độ cao (GPU, SSD NVMe, Card mạng). Sử dụng các cặp tín hiệu vi sai xung nhịp:

- **CKP** (Clock Positive): Cực dương xung đồng hồ.
- **CKM** (Clock Minus): Cực âm xung đồng hồ.

Cặp tín hiệu vi sai thường thấy trong ``pcie_<>`` (hậu tố của PCIe).

---

.. rubric:: 5. Tín Hiệu Âm Thanh & Hình Ảnh (Audio & Video)

**Audio:**

- **ADAC_AOR:** Audio DAC Output Right (Kênh Phải).
- **ADAC_AOL:** Audio DAC Output Left (Kênh Trái).

**Video CVBS (Composite Video Blanking and Sync):**

- **CVBS_OUT:** Đường xuất video Analog ra jack AV/RCA màu vàng.
- **C (Composite):** Tổng hợp dữ liệu hình ảnh, màu sắc và xung đồng bộ trên 1 dây.
- **V (Video):** Tín hiệu hình ảnh.
- **B (Blanking):** Xung trống dập tia.
- **S (Sync):** Xung đồng bộ hàng/khung hình (chống xé/cuộn hình).

---

.. rubric:: Tài Liệu Tham Khảo

- `Banana Pi Official Website <https://www.banana-pi.org/>`_
- `Banana Pi M4 Manual <https://wiki.banana-pi.org/Banana_Pi_BPI-M4>`_
- `Reading Schematics Guide <https://learn.sparkfun.com/tutorials/how-to-read-a-schematic>`_