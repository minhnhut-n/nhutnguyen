Level 2 – Embedded Linux
========================

Board của bạn rất phù hợp cho việc học embedded Linux vì Banana Pi sử dụng
Allwinner SoC, có đầy đủ các khối ngoại vi như GPIO, UART, I2C, SPI, Ethernet
và quan trọng nhất là tài liệu open-source rất phong phú.

Boot
----

- BootROM
- SPL
- U-Boot
- Device Tree
- Kernel
- init
- systemd

Hiểu toàn bộ quá trình boot là bước đầu tiên để làm chủ embedded Linux. Bạn
sẽ biết chuyện gì xảy ra từ khi nhấn nút nguồn cho đến khi bạn thấy màn hình
login.

**1. BootROM**

BootROM là một đoạn code nhỏ được nhà sản xuất SoC (System-on-Chip) ghi sẵn
vào ROM (Read-Only Memory) của chip. Nó là thứ đầu tiên chạy khi bạn cấp
nguồn cho board. Nhiệm vụ chính là xác định boot device (SD card, eMMC,
NAND Flash, UART, USB,...) và load SPL vào SRAM.

.. code-block:: bash

   # BootROM thường dùng FEL mode trên Allwinner để boot qua USB
   # Có thể kiểm tra FEL mode bằng cách:
   $ lsusb
   # Nếu thấy "1f3a:efe8" (Onda) hoặc tương tự, board đang ở FEL mode

**2. SPL (Secondary Program Loader)**

SPL là bootloader giai đoạn 2, được load bởi BootROM vào SRAM (vì DRAM chưa
được khởi tạo). Nhiệm vụ của SPL là:

- Khởi tạo clock cơ bản
- Khởi tạo DRAM (bộ nhớ chính)
- Load U-Boot từ boot device vào DRAM
- Jump sang U-Boot

.. code-block:: bash

   # SPL thường là file "sunxi-spl.bin" trên các board Allwinner
   # Xem SPL trong file image:
   $ hexdump -C u-boot-sunxi-with-spl.bin | head -20

**3. U-Boot (Universal Bootloader)**

U-Boot là bootloader phổ biến nhất trong thế giới embedded Linux. Nó là một
"mini OS" có shell riêng, cho phép bạn tương tác trước khi kernel được load.
U-Boot làm các nhiệm vụ sau:

- Khởi tạo hardware (Ethernet, MMC, USB,...)
- Đọc Device Tree và Kernel từ storage
- Cung cấp môi trường (environment) để cấu hình boot parameters
- Load và jump vào kernel

.. code-block:: bash

   # Vào U-Boot console (nhấn phím bất kỳ khi boot)
   U-Boot> printenv           # Xem các biến môi trường
   U-Boot> mmc list           # Xem danh sách MMC devices
   U-Boot> mmc dev 0          # Chọn MMC device 0
   U-Boot> fatls mmc 0:1      # Liệt kê file trong partition 1 (FAT)
   U-Boot> bdinfo             # Thông tin board
   U-Boot> boot               # Boot kernel

   # Boot kernel bằng tay:
   U-Boot> fatload mmc 0:1 0x42000000 uImage
   U-Boot> fatload mmc 0:1 0x43000000 sun7i-a20-bananapi.dtb
   U-Boot> bootm 0x42000000 - 0x43000000

**4. Device Tree (Flattened Device Tree - FDT)**

Device Tree là một cấu trúc dữ liệu dạng cây (tree) mô tả toàn bộ phần cứng
trên board: CPU, memory, GPIO, UART, I2C, interrupt controller,... 

Tại sao cần Device Tree? Hồi xưa (kernel 2.6 trở về trước), thông tin phần
cứng được hardcode trong source code kernel. Mỗi board là một file ``board-*.c``
riêng. Hàng ngàn board ⇒ không thể maintain nổi. Device Tree ra đời để tách
phần mô tả hardware ra khỏi kernel code.

.. code-block:: dts

   // Ví dụ Device Tree cho Banana Pi (sun7i-a20-bananapi.dts)
   /dts-v1/;

   #include "sun7i-a20.dtsi"

   / {
       model = "Banana Pi";
       compatible = "bananapi,bpi-m1", "allwinner,sun7i-a20";

       memory {
           reg = <0x40000000 0x80000000>;  // 2GB RAM
       };

       leds {
           compatible = "gpio-leds";
           pinctrl-0 = <&led_pins>;

           led-green {
               label = "bananapi:green:usr";
               gpios = <&pio 7 24 GPIO_ACTIVE_HIGH>;
               default-state = "on";
           };
       };
   };

   &uart0 {
       pinctrl-0 = <&uart0_pins>;
       status = "okay";
   };

   &mmc0 {
       pinctrl-0 = <&mmc0_pins>;
       vmmc-supply = <&reg_vcc3v3>;
       bus-width = <4>;
       non-removable;
       status = "okay";
   };

Device Tree compiler (DTC) sẽ biên dịch ``.dts`` thành ``.dtb`` (binary).
Kernel đọc ``.dtb`` để biết nó đang chạy trên hardware nào và khởi tạo driver
tương ứng.

.. code-block:: bash

   # Biên dịch DTS → DTB
   $ dtc -I dts -O dtb -o sun7i-a20-bananapi.dtb sun7i-a20-bananapi.dts

   # Giải ngược DTB → DTS (để xem)
   $ dtc -I dtb -O dts -o output.dts sun7i-a20-bananapi.dtb

   # Xem Device Tree của hệ thống đang chạy
   $ ls /sys/firmware/devicetree/base/
   $ cat /sys/firmware/devicetree/base/model  # Xem model name

**5. Kernel**

Kernel là trái tim của hệ điều hành Linux. Nó quản lý memory, processes,
drivers, filesystem, networking,... U-Boot load kernel image vào memory và
nhảy đến entry point. Kernel khởi tạo các subsystem, đọc Device Tree để
probe drivers, mount root filesystem và cuối cùng chạy ``/sbin/init``.

.. code-block:: bash

   # Kernel image thường là zImage, uImage hoặc Image
   # Xem kernel version đang chạy
   $ uname -a
   $ cat /proc/version

   # Xem kernel modules đã load
   $ lsmod

   # Xem kernel log (dmesg)
   $ dmesg | head -50  # 50 dòng đầu tiên từ kernel ring buffer

   # Kiểm tra kernel config
   $ zcat /proc/config.gz

**6. init**

Init là process đầu tiên được kernel chạy (PID = 1). Nó là "tổ tiên" của
mọi process khác trên hệ thống. Trên các hệ thống embedded nhỏ gọn, init
có thể là BusyBox init hoặc một init script đơn giản.

.. code-block:: bash

   # Kiểm tra PID 1
   $ ps -p 1 -o pid,comm,cmd

   # Trên hệ thống dùng BusyBox, init thường là /sbin/init
   $ ls -l /sbin/init

   # Init script mặc định: /etc/inittab (dùng cho BusyBox init)
   $ cat /etc/inittab

**7. systemd**

Systemd là init system hiện đại, được dùng trên hầu hết các distro Linux
hiện nay (Ubuntu, Debian, Fedora,...). Nó không chỉ là init mà còn quản lý
service, logging (journald), network (networkd), timer (thay cron), v.v.

.. code-block:: bash

   # systemd là PID 1 trên hầu hết distro hiện đại
   $ systemctl status
   $ systemctl list-units --type=service  # Liệt kê services
   $ systemctl get-default                  # Default target (multi-user/graphical)
   $ systemctl isolate multi-user.target    # Chuyển sang chế độ không GUI

   # Phân tích thời gian boot
   $ systemd-analyze
   $ systemd-analyze blame                 # Xem service nào boot lâu nhất

   # Xem log của một service
   $ journalctl -u ssh.service

Storage
-------

- MMC
- SD
- eMMC
- Partition
- mount
- fstab

**1. MMC (MultiMediaCard)**

MMC là chuẩn giao tiếp cho thẻ nhớ, dùng giao thức serial 1-bit hoặc
multi-bit (4-bit, 8-bit). Trên Banana Pi, MMC controller kết nối với
khe SD card và eMMC chip.

.. code-block:: bash

   # Xem MMC devices
   $ ls /dev/mmcblk*

   # Thông tin chi tiết về MMC device
   $ cat /sys/block/mmcblk0/device/name
   $ cat /sys/block/mmcblk0/device/manfid

   # Xem kernel log về MMC
   $ dmesg | grep mmc

**2. SD (Secure Digital)**

SD là phiên bản phát triển của MMC, phổ biến trên các board nhúng và
máy ảnh. SD card có thể dùng làm boot device cho Banana Pi.

.. code-block:: bash

   # Kiểm tra tốc độ SD card
   $ dd if=/dev/mmcblk0 of=/dev/null bs=1M count=100
   # Kết quả cho thấy tốc độ đọc (càng cao càng tốt)

   # Ghi image vào SD card (cẩn thận với of=)
   $ sudo dd if=ubuntu.img of=/dev/mmcblk0 bs=4M status=progress

**3. eMMC (embedded MMC)**

eMMC là MMC được hàn cố định trên board (không thể tháo rời như SD card).
eMMC nhanh hơn SD card và tin cậy hơn. Banana Pi Pro có eMMC onboard.

.. code-block:: bash

   # Kiểm tra dung lượng eMMC
   $ fdisk -l /dev/mmcblk1   # eMMC thường là mmcblk1

   # Xem eMMC health (nếu driver hỗ trợ)
   $ cat /sys/block/mmcblk1/device/life_time
   $ cat /sys/block/mmcblk1/device/pre_eol_info

**4. Partition**

Partition là cách chia ổ đĩa thành các vùng riêng biệt. Trên embedded Linux,
bạn thường thấy layout kiểu:

.. code-block:: bash

   # Layout điển hình trên Banana Pi boot từ SD card
   $ fdisk -l /dev/mmcblk0

   # Ví dụ output:
   # Device         Boot  Start      End  Sectors  Size Id Type
   # /dev/mmcblk0p1 *      2048   133119   131072   64M  c W95 FAT32 (boot)
   # /dev/mmcblk0p2      133120 15564799 15431680  7.4G 83 Linux (rootfs)

- Partition 1 (FAT32): Chứa bootloader, kernel, device tree
- Partition 2 (ext4): Chứa root filesystem (Ubuntu/Debian)

Công cụ partition phổ biến:

.. code-block:: bash

   # fdisk - cổ điển, ổn định
   $ sudo fdisk /dev/mmcblk0

   # parted - hỗ trợ GPT
   $ sudo parted /dev/mmcblk0 print

   # gdisk - dành cho GPT
   $ sudo gdisk /dev/mmcblk0

**5. mount**

Mount là quá trình gắn một filesystem vào một thư mục (mount point) để
truy cập dữ liệu. Không mount thì không xài được.

.. code-block:: bash

   # Mount thủ công
   $ sudo mount /dev/mmcblk0p2 /mnt/rootfs
   $ ls /mnt/rootfs

   # Mount với options
   $ sudo mount -o rw,noatime /dev/mmcblk0p1 /mnt/boot

   # Xem tất cả mount points
   $ mount
   $ df -hT

   # Unmount
   $ sudo umount /mnt/rootfs

**6. fstab (File System Table)**

File ``/etc/fstab`` là file cấu hình cho biết partition nào sẽ được mount
tự động vào thư mục nào khi boot.

.. code-block:: bash

   # Ví dụ /etc/fstab trên Banana Pi
   $ cat /etc/fstab

   # Format:
   # <device>    <mount_point>  <fstype>  <options>  <dump>  <pass>
   /dev/mmcblk0p1  /boot         vfat      defaults    0       2
   /dev/mmcblk0p2  /             ext4      defaults,noatime 0 1
   tmpfs           /tmp          tmpfs     defaults    0       0

Giải thích các cột:

- **device**: Block device (``/dev/mmcblk0p1``) hoặc UUID
- **mount_point**: Thư mục mount (``/boot``, ``/``)
- **fstype**: Loại filesystem (ext4, vfat, tmpfs)
- **options**: ``defaults``, ``noatime`` (tăng tốc), ``ro`` (read-only)
- **dump**: Có backup bằng dump không (0 = không)
- **pass**: Thứ tự kiểm tra fsck (1 = root, 2 = other, 0 = không kiểm tra)

Dùng UUID để tránh lỗi khi thay đổi device name:

.. code-block:: bash

   # Lấy UUID của partition
   $ blkid /dev/mmcblk0p2
   # Output: /dev/mmcblk0p2: UUID="abc123..." TYPE="ext4"

   # /etc/fstab với UUID
   UUID=abc123... / ext4 defaults,noatime 0 1

Networking
----------

- Ethernet
- Wi-Fi
- Bluetooth
- DHCP
- DNS
- SSH
- FTP
- NFS

**1. Ethernet**

Banana Pi có cổng Ethernet 10/100/1000 Mbps (tùy model) sử dụng chip
Realtek hoặc integrated MAC + external PHY.

.. code-block:: bash

   # Kiểm tra interface
   $ ip link show
   $ ifconfig -a

   # Bật/tắt Ethernet
   $ sudo ip link set eth0 up
   $ sudo ip link set eth0 down

   # Gán IP tĩnh
   $ sudo ip addr add 192.168.1.100/24 dev eth0
   $ sudo ip route add default via 192.168.1.1

   # Xem thông tin chi tiết Ethernet (link speed, duplex)
   $ ethtool eth0

   # Test kết nối
   $ ping -c 4 google.com

**2. Wi-Fi**

Wi-Fi trên Banana Pi thường dùng chip USB Wi-Fi hoặc module onboard (như
AP6210 trên Banana Pi Pro).

.. code-block:: bash

   # Quét mạng Wi-Fi
   $ sudo iwlist wlan0 scan

   # Kết nối với WPA/WPA2
   $ sudo nano /etc/wpa_supplicant/wpa_supplicant.conf
   # Thêm:
   # network={
   #     ssid="TEN_WIFI"
   #     psk="mat_khau"
   # }

   $ sudo wpa_supplicant -B -i wlan0 -c /etc/wpa_supplicant/wpa_supplicant.conf
   $ sudo dhclient wlan0  # Lấy IP

   # Hoặc dùng nmcli (NetworkManager) cho dễ
   $ sudo nmcli dev wifi connect "TEN_WIFI" password "mat_khau"

**3. Bluetooth**

Bluetooth trên Banana Pi thường đi kèm với module Wi-Fi (combo chip).

.. code-block:: bash

   # Kiểm tra Bluetooth adapter
   $ hciconfig -a
   $ bluetoothctl

   # Scan devices
   $ bluetoothctl scan on

   # Pair với device
   $ bluetoothctl pair <MAC_ADDR>
   $ bluetoothctl connect <MAC_ADDR>

**4. DHCP (Dynamic Host Configuration Protocol)**

DHCP tự động cấp IP address, gateway, DNS cho client. Trên embedded Linux,
DHCP client phổ biến là ``dhclient`` (ISC) hoặc ``dhcpcd`` (nhẹ hơn).

.. code-block:: bash

   # Lấy IP tự động từ DHCP server
   $ sudo dhclient eth0

   # Renew IP
   $ sudo dhclient -r eth0   # Release
   $ sudo dhclient eth0      # Renew

   # Cấu hình DHCP tĩnh trong /etc/dhcpcd.conf
   $ cat /etc/dhcpcd.conf
   # interface eth0
   # static ip_address=192.168.1.100/24
   # static routers=192.168.1.1
   # static domain_name_servers=8.8.8.8

**5. DNS (Domain Name System)**

DNS chuyển đổi tên miền (google.com) thành IP address. File cấu hình DNS
là ``/etc/resolv.conf``.

.. code-block:: bash

   # Xem DNS servers
   $ cat /etc/resolv.conf

   # Test DNS resolution
   $ nslookup google.com
   $ dig google.com
   $ host google.com

   # Nếu không có DNS, bạn có thể ping trực tiếp bằng IP:
   $ ping 8.8.8.8  # Google DNS

**6. SSH (Secure Shell)**

SSH là giao thức "cứu cánh" cho embedded Linux vì board thường không có
màn hình. Bạn SSH vào board từ máy tính để làm việc.

.. code-block:: bash

   # Trên board Banana Pi - cài SSH server
   $ sudo apt-get install openssh-server
   $ sudo systemctl enable ssh
   $ sudo systemctl start ssh

   # Kiểm tra SSH đang chạy
   $ sudo systemctl status ssh
   $ ss -tulpn | grep :22

   # Từ máy tính - SSH vào board
   $ ssh pi@192.168.1.100
   # Mặc định password: bananapi

   # Copy SSH key để không cần nhập password
   $ ssh-copy-id pi@192.168.1.100

   # Config SSH cho bảo mật (không cho phép root login)
   $ sudo nano /etc/ssh/sshd_config
   # PermitRootLogin no

**7. FTP (File Transfer Protocol)**

FTP dùng để truyền file giữa máy tính và board. Tuy nhiên, FTP không mã
hóa nên ngày nay ít dùng, thay bằng SFTP (SSH-based).

.. code-block:: bash

   # Cài FTP server (vsftpd)
   $ sudo apt-get install vsftpd
   $ sudo systemctl start vsftpd

   # Hoặc dùng SFTP (qua SSH) - khuyên dùng
   $ sftp pi@192.168.1.100
   sftp> put localfile.txt
   sftp> get remotefile.txt
   sftp> ls
   sftp> exit

**8. NFS (Network File System)**

NFS cho phép mount một thư mục từ máy tính (host) vào board (target) qua
mạng. Rất hữu ích cho development: bạn edit code trên máy tính, board chạy
trực tiếp mà không cần copy file.

.. code-block:: bash

   # Trên máy tính (host) - share thư mục
   $ sudo apt-get install nfs-kernel-server
   $ sudo nano /etc/exports
   # /path/to/rootfs 192.168.1.0/24(rw,no_root_squash,no_subtree_check)
   $ sudo exportfs -a
   $ sudo systemctl restart nfs-kernel-server

   # Trên board Banana Pi - mount NFS
   $ sudo mount -t nfs 192.168.1.100:/path/to/rootfs /mnt/nfs
   $ ls /mnt/nfs

   # NFS root boot (kernel boot thẳng từ NFS, không cần SD card)
   # Cấu hình trong U-Boot:
   # setenv bootargs console=ttyS0,115200 root=/dev/nfs nfsroot=192.168.1.100:/path/to/rootfs ip=192.168.1.100

GPIO
----

- GPIO sysfs
- interrupt
- LED
- Button

**1. GPIO sysfs**

GPIO sysfs là interface cũ (nhưng đơn giản) để điều khiển GPIO từ userspace.
Các file nằm trong ``/sys/class/gpio/``. Này đã bị deprecated từ kernel 4.8,
nhưng vẫn còn được dùng trong các project cũ hoặc prototype nhanh.

.. code-block:: bash

   # Export GPIO pin (ví dụ GPIO 7 trên Banana Pi)
   $ echo 7 > /sys/class/gpio/export
   $ ls /sys/class/gpio/gpio7/

   # Cấu hình direction (in/out)
   $ echo out > /sys/class/gpio/gpio7/direction
   $ echo in > /sys/class/gpio/gpio7/direction

   # Ghi giá trị (output)
   $ echo 1 > /sys/class/gpio/gpio7/value  # HIGH (3.3V)
   $ echo 0 > /sys/class/gpio/gpio7/value  # LOW (0V)

   # Đọc giá trị (input)
   $ cat /sys/class/gpio/gpio7/value

   # Unexport khi không dùng nữa
   $ echo 7 > /sys/class/gpio/unexport

.. attention::

   GPIO sysfs cũ, chậm và không real-time. Trong kernel mới, dùng
   ``libgpiod`` (character device interface) thay thế. Nhưng sysfs vẫn
   dễ cho người mới học.

**libgpiod** - GPIO character device interface mới:

.. code-block:: bash

   # Kiểm tra gpiochip
   $ gpiodetect
   # gpiochip0 [sun7i-a20-pinctrl] (128 lines)

   # Xem thông tin từng pin
   $ gpioinfo gpiochip0

   # Set pin 7 làm output và ghi value 1
   $ gpioset gpiochip0 7=1
   $ gpioset gpiochip0 7=0

   # Đọc input
   $ gpioget gpiochip0 7

   # Monitor chờ sự kiện trên pin
   $ gpiomon gpiochip0 7

**2. interrupt (GPIO Interrupt)**

GPIO interrupt cho phép bạn "chờ" sự kiện trên một pin mà không cần
polling (vừa tốn CPU vừa chậm). Khi pin thay đổi trạng thái, kernel
gửi signal đến userspace.

.. code-block:: bash

   # Trên sysfs, config interrupt:
   $ echo 7 > /sys/class/gpio/export
   $ echo in > /sys/class/gpio/gpio7/direction
   $ echo both > /sys/class/gpio/gpio7/edge
   # edge: rising, falling, both

   # Dùng poll() trong C để chờ sự kiện
   # (Xem lại level 1 - poll() system call)

.. code-block:: c

   // GPIO interrupt với libgpiod (C)
   #include <gpiod.h>
   #include <stdio.h>
   #include <unistd.h>

   int main() {
       struct gpiod_chip *chip = gpiod_chip_open_by_name("gpiochip0");
       struct gpiod_line *line = gpiod_chip_get_line(chip, 7);

       // Cấu hình input với falling edge interrupt
       gpiod_line_request_falling_edge_events(line, "myapp");

       struct gpiod_line_bulk bulk;
       gpiod_line_bulk_init(&bulk);
       gpiod_line_bulk_add(&bulk, line);

       while (1) {
           // Chờ sự kiện (blocking)
           int ret = gpiod_line_event_wait_bulk(&bulk, NULL, NULL);
           if (ret > 0) {
               struct gpiod_line_event event;
               gpiod_line_event_read(line, &event);
               printf("Interrupt! Timestamp: %ld.%ld\n",
                      event.ts.tv_sec, event.ts.tv_nsec);
           }
       }
   }

**3. LED**

LED là ứng dụng cơ bản nhất của GPIO output. Nhấp nháy LED là "Hello World"
của embedded Linux.

.. code-block:: bash

   # Cách 1: GPIO sysfs (thủ công)
   $ echo 24 > /sys/class/gpio/export      # GPIO 24 (PH24 trên Banana Pi)
   $ echo out > /sys/class/gpio/gpio24/direction
   $ echo 1 > /sys/class/gpio/gpio24/value  # Bật LED
   $ sleep 1
   $ echo 0 > /sys/class/gpio/gpio24/value  # Tắt LED

   # Cách 2: LED class driver (nếu Device Tree đã config)
   $ echo 1 > /sys/class/leds/bananapi:green:usr/brightness
   $ echo 0 > /sys/class/leds/bananapi:green:usr/brightness

   # Chế độ nhấp nháy (trigger)
   $ cat /sys/class/leds/bananapi:green:usr/trigger
   $ echo heartbeat > /sys/class/leds/bananapi:green:usr/trigger

.. code-block:: c

   // Blinky LED với C
   #include <stdio.h>
   #include <stdlib.h>
   #include <unistd.h>
   #include <fcntl.h>

   int main() {
       int fd = open("/sys/class/gpio/gpio24/value", O_WRONLY);
       if (fd < 0) {
           perror("open GPIO failed");
           return -1;
       }

       while (1) {
           write(fd, "1", 1);  // Bật
           usleep(500000);     // 500ms
           write(fd, "0", 1);  // Tắt
           usleep(500000);     // 500ms
       }

       close(fd);
       return 0;
   }

**4. Button**

Button là ứng dụng cơ bản của GPIO input, thường dùng để reset config,
shutdown, hoặc trigger một hành động.

.. code-block:: bash

   # Đọc trạng thái button (polling)
   $ echo 7 > /sys/class/gpio/export
   $ echo in > /sys/class/gpio/gpio7/direction

   while true; do
       val=$(cat /sys/class/gpio/gpio7/value)
       if [ "$val" = "1" ]; then
           echo "Button pressed!"
       fi
       sleep 0.1
   done

   # Dùng gpio-keys driver (config trong Device Tree)
   # Kernel tự xử lý debounce và gửi sự kiện input

.. code-block:: dts

   // Device Tree cho button
   gpio-keys {
       compatible = "gpio-keys";
       #address-cells = <1>;
       #size-cells = <0>;

       button@0 {
           label = "SW1";
           linux,code = <KEY_POWER>;  // Mã key (keycodes)
           gpios = <&pio 7 7 GPIO_ACTIVE_LOW>;  // PH7, active low
           debounce-interval = <50>;  // 50ms debounce
       };
   };

.. code-block:: bash

   # Khi button được nhấn, kernel gửi sự kiện input
   $ evtest
   # Chọn /dev/input/eventX tương ứng
   # Nhấn button và xem output

UART
----

- serial
- DMA
- baudrate
- flow control

**1. Serial (UART)**

UART (Universal Asynchronous Receiver/Transmitter) là giao thức nối tiếp
cổ điển, dùng để debug console, giao tiếp với module GPS, GSM, LoRa,...
Trên Banana Pi, UART0 thường là debug console (qua pin header).

.. code-block:: bash

   # Xem UART devices
   $ ls /dev/ttyS* /dev/ttyAMA*

   # Banana Pi thường có:
   # /dev/ttyS0 - UART0 (debug console, qua GPIO header)
   # /dev/ttyS1 - UART1
   # /dev/ttyS2 - UART2

   # Kiểm tra baudrate và config
   $ stty -F /dev/ttyS0 -a

   # Gửi dữ liệu qua UART
   $ echo "Hello" > /dev/ttyS0

   # Đọc dữ liệu từ UART
   $ cat /dev/ttyS0

   # Kết nối với serial terminal
   $ sudo apt-get install picocom minicom
   $ picocom -b 115200 /dev/ttyS0
   # Ctrl+A Ctrl+X để thoát picocom

.. code-block:: c

   // Lập trình UART với C
   #include <stdio.h>
   #include <fcntl.h>
   #include <unistd.h>
   #include <termios.h>

   int main() {
       int fd = open("/dev/ttyS0", O_RDWR | O_NOCTTY);
       if (fd < 0) {
           perror("open UART failed");
           return -1;
       }

       struct termios tty;
       tcgetattr(fd, &tty);

       // Set baudrate 115200
       cfsetospeed(&tty, B115200);
       cfsetispeed(&tty, B115200);

       tty.c_cflag |= (CLOCAL | CREAD);    // Enable receiver
       tty.c_cflag &= ~CSIZE;
       tty.c_cflag |= CS8;                  // 8 data bits
       tty.c_cflag &= ~PARENB;              // No parity
       tty.c_cflag &= ~CSTOPB;              // 1 stop bit
       tty.c_cflag &= ~CRTSCTS;             // No hardware flow control

       tty.c_lflag &= ~(ICANON | ECHO | ECHOE | ISIG);  // Raw mode
       tty.c_iflag &= ~(IXON | IXOFF | IXANY);          // No software flow control
       tty.c_oflag &= ~OPOST;                            // Raw output

       tcsetattr(fd, TCSANOW, &tty);

       // Gửi dữ liệu
       write(fd, "AT\r\n", 4);

       // Đọc dữ liệu
       char buf[256];
       int n = read(fd, buf, sizeof(buf));
       if (n > 0) {
           buf[n] = '\0';
           printf("Received: %s\n", buf);
       }

       close(fd);
       return 0;
   }

**2. DMA (Direct Memory Access)**

DMA cho phép UART truyền/nhận dữ liệu trực tiếp với RAM mà không cần CPU
tham gia từng byte. CPU chỉ cần setup và nhận interrupt khi hoàn thành.
Điều này cực kỳ quan trọng cho các ứng dụng cần throughput cao.

.. code-block:: bash

   # Kiểm tra DMA có được enable không
   $ dmesg | grep dma
   $ ls /sys/class/dma/

   # Xem DMA channels đang dùng
   $ cat /sys/kernel/debug/dmaengine/summary

   # Trên Banana Pi, DMA controller là "sun4i-dma"
   $ cat /proc/device-tree/soc/dma-controller@* 2>/dev/null

.. note::

   DMA trên UART thường được kernel driver quản lý tự động. Bạn không cần
   config gì thêm ở userspace. Nhưng nếu dùng UART ở chế độ polling
   (không DMA), CPU sẽ bận rộn cho mỗi byte, rất lãng phí.

**3. Baudrate**

Baudrate là tốc độ truyền dữ liệu qua UART, đơn vị là bits per second (bps).
Các baudrate phổ biến:

- ``9600`` - Tiêu chuẩn cũ, dùng cho terminal cổ điển
- ``115200`` - Phổ biến nhất cho embedded Linux debug console
- ``230400`` - Nhanh hơn
- ``921600`` - Rất nhanh, dùng khi cần throughput cao

.. code-block:: bash

   # Kiểm tra baudrate hiện tại
   $ stty -F /dev/ttyS0 speed

   # Đổi baudrate (trên terminal đã mở)
   $ stty -F /dev/ttyS0 115200

   # Custom baudrate (kernel hỗ trợ)
   $ stty -F /dev/ttyS0 1500000  # 1.5 Mbps

.. warning::

   Baudrate giữa hai thiết bị UART phải giống nhau, nếu không dữ liệu
   sẽ bị lỗi (garbled). Ví dụ nếu board gửi ở 115200 mà terminal nhận
   ở 9600, bạn sẽ thấy toàn ký tự rác.

**4. Flow Control (Lưu lượng điều khiển)**

Flow control ngăn buffer tràn khi một bên gửi nhanh hơn bên kia xử lý.

Có hai loại:

- **Hardware flow control (RTS/CTS)**: Dùng 2 dây riêng (Request To Send /
  Clear To Send) để báo hiệu. Đáng tin cậy, dùng trong modem, GPS modules.
- **Software flow control (XON/XOFF)**: Dùng ký tự đặc biệt (Ctrl+S = XOFF,
  Ctrl+Q = XON) trong data stream. Đơn giản nhưng dễ bị lỗi với binary data.

.. code-block:: bash

   # Bật hardware flow control
   $ stty -F /dev/ttyS0 crtscts

   # Tắt flow control
   $ stty -F /dev/ttyS0 -crtscts

   # Bật software flow control
   $ stty -F /dev/ttyS0 ixon ixoff

.. code-block:: c

   // Cấu hình flow control trong C
   struct termios tty;
   tcgetattr(fd, &tty);

   // Enable hardware flow control (RTS/CTS)
   tty.c_cflag |= CRTSCTS;

   // Disable hardware flow control
   tty.c_cflag &= ~CRTSCTS;

   // Enable software flow control (XON/XOFF)
   tty.c_iflag |= (IXON | IXOFF);

   tcsetattr(fd, TCSANOW, &tty);

Lưu ý: Khi dùng hardware flow control, bạn cần kết nối đúng dây RTS và CTS
giữa hai thiết bị. Nếu không có dây này, tắt flow control để tránh bị treo
(hang) khi một bên chờ tín hiệu RTS/CTS không bao giới đến.

.. include:: ../../../../_includes/contact_info.rst