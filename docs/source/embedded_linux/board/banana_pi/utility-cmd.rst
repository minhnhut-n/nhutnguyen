3. Utility Commands
=============================================================================

.. rubric:: Giới thiệu

Tổng hợp các câu lệnh hữu ích cho hệ thống Linux nhúng (Embedded Linux). Các lệnh được phân loại theo từng nhóm chức năng để dễ tra cứu.

---

.. rubric:: 1. System (Thông tin hệ thống)

.. code-block:: bash

   # Xem thông tin nhân & hệ thống
   uname -a
   hostnamectl
   cat /etc/os-release

   # Xem thời gian hoạt động & ngày giờ
   uptime
   date
   timedatectl

   # Xem thông tin CPU, RAM, ổ đĩa
   lscpu
   free -h
   lsblk
   df -h
   mount

---

.. rubric:: 2. Services (Quản lý dịch vụ)

.. code-block:: bash

   # Xem các service bị lỗi
   systemctl --failed

   # Xem trạng thái service
   systemctl status <service>

   # Quản lý service
   systemctl start|stop|restart|enable|disable <service>

   # Xem log service trong lần boot hiện tại
   journalctl -u <service> -b

---

.. rubric:: 3. Boot (Quản lý khởi động)

.. code-block:: bash

   # Xem target mặc định hiện tại
   systemctl get-default

   # Chuyển sang chế độ không GUI (multi-user)
   systemctl set-default multi-user.target

   # Chuyển sang chế độ có GUI (graphical)
   systemctl set-default graphical.target

   # Áp dụng ngay lập tức (không cần reboot)
   systemctl isolate multi-user.target
   systemctl isolate graphical.target

---

.. rubric:: 4. Network (Mạng)

.. code-block:: bash

   # Xem địa chỉ IP và giao diện mạng
   ip addr
   ip route

   # Xem các cổng đang lắng nghe
   ss -tulpn

   # Xem IP hiện tại
   hostname -I

   # Xem trạng thái các thiết bị mạng (NetworkManager)
   nmcli device status

---

.. rubric:: 5. WiFi

.. code-block:: bash

   # Bật/tắt WiFi
   nmcli radio wifi on
   nmcli radio wifi off

   # Quét các mạng WiFi khả dụng
   nmcli device wifi list

   # Kết nối WiFi
   nmcli device wifi connect "SSID" password "PASSWORD"

---

.. rubric:: 6. Bluetooth

.. code-block:: bash

   # Quản lý service bluetooth
   systemctl status bluetooth
   systemctl start bluetooth
   systemctl stop bluetooth

   # Giao diện dòng lệnh bluetooth
   bluetoothctl
   # Các lệnh trong bluetoothctl:
   #   power on/off       - Bật/tắt Bluetooth
   #   scan on/off        - Quét thiết bị
   #   devices            - Danh sách thiết bị đã ghép đôi
   #   pair <MAC>         - Ghép đôi với thiết bị
   #   connect <MAC>      - Kết nối đến thiết bị
   #   remove <MAC>       - Xóa thiết bị

---

.. rubric:: 7. SSH

.. code-block:: bash

   # Xem trạng thái SSH
   systemctl status ssh

   # Bật và khởi động SSH
   systemctl enable ssh
   systemctl start ssh

   # Kiểm tra cổng SSH đang lắng nghe
   ss -tlnp | grep ssh

---

.. rubric:: 8. Kernel (Quản lý module nhân)

.. code-block:: bash

   # Xem phiên bản kernel
   uname -r

   # Xem danh sách module đã nạp
   lsmod

   # Xem thông tin module
   modinfo <module>

   # Nạp/gỡ module
   modprobe <module>           # Nạp module
   modprobe -r <module>        # Gỡ module
   insmod file.ko              # Nạp module từ file
   rmmod module                # Gỡ module

   # Xem log kernel (real-time)
   dmesg -w

---

.. rubric:: 9. USB & I2C

.. code-block:: bash

   # Xem danh sách thiết bị USB
   lsusb
   lsusb -t                    # Xem dạng cây

   # Quét bus I2C
   i2cdetect -l                # Liệt kê các bus I2C
   i2cdetect -y 1              # Quét thiết bị trên bus 1

---

.. rubric:: 10. Disk (Định dạng & Mount ổ đĩa với fdisk)

.. code-block:: bash

   # -----------------------------------------------------------
   # 1. Xem danh sách ổ đĩa
   # -----------------------------------------------------------
   lsblk                     # Xem cây block devices
   fdisk -l                  # Xem chi tiết partition table
   blkid                     # Xem UUID và filesystem type

   # -----------------------------------------------------------
   # 2. Phân vùng ổ đĩa với fdisk
   # -----------------------------------------------------------
   # Mở fdisk cho device (vd: /dev/sda, /dev/mmcblk0)
   sudo fdisk /dev/sda

   # Các lệnh trong fdisk:
   #   m    - Xem help
   #   p    - In partition table
   #   n    - Tạo partition mới
   #   d    - Xóa partition
   #   t    - Đổi type partition (Linux=83, FAT32=b, Swap=82)
   #   w    - Ghi thay đổi và thoát
   #   q    - Thoát không lưu

   # Ví dụ: Tạo 2 partition trên SD card 8GB (/dev/mmcblk0)
   # Bước 1: Mở fdisk
   sudo fdisk /dev/mmcblk0

   # Bước 2: Xoá partition cũ (nếu có)
   #   Lệnh: d
   #   Lặp lại nếu có nhiều partition

   # Bước 3: Tạo partition 1 - FAT32 (boot, 64MB)
   #   Lệnh: n → p (primary) → 1 → (enter) → +64M
   #   Lệnh: t → b (W95 FAT32)

   # Bước 4: Tạo partition 2 - Linux (rootfs, phần còn lại)
   #   Lệnh: n → p → 2 → (enter) → (enter)
   #   Lệnh: t → 2 → 83 (Linux)

   # Bước 5: Ghi lại
   #   Lệnh: w

   # Xác nhận sau khi ghi:
   sudo fdisk -l /dev/mmcblk0

   # -----------------------------------------------------------
   # 3. Định dạng filesystem (format)
   # -----------------------------------------------------------
   # Định dạng ext4 (cho rootfs)
   sudo mkfs.ext4 /dev/mmcblk0p2

   # Định dạng FAT32 (cho boot partition)
   sudo mkfs.vfat -F 32 /dev/mmcblk0p1

   # Định dạng NTFS (dùng cho USB HDD tương thích Windows)
   sudo mkfs.ntfs /dev/sda1

   # Định dạng Swap
   sudo mkswap /dev/mmcblk0p3
   sudo swapon /dev/mmcblk0p3   # Bật swap ngay lập tức

   # Định dạng nhanh (không kiểm tra bad block)
   sudo mkfs.ext4 -F /dev/mmcblk0p2

   # Định dạng với label
   sudo mkfs.ext4 -L rootfs /dev/mmcblk0p2
   sudo mkfs.vfat -F 32 -n BOOT /dev/mmcblk0p1

   # -----------------------------------------------------------
   # 4. Mount ổ đĩa
   # -----------------------------------------------------------
   # Mount thủ công
   sudo mount /dev/mmcblk0p2 /mnt
   sudo mount /dev/mmcblk0p1 /mnt/boot

   # Mount với options tối ưu cho embedded
   sudo mount -o noatime,nodiratime,data=writeback /dev/mmcblk0p2 /mnt

   # Mount tất cả partition trong fstab
   sudo mount -a

   # Mount với UUID (tránh lỗi khi device name thay đổi)
   sudo mount UUID=abc123... /mnt

   # Unmount
   sudo umount /mnt

   # -----------------------------------------------------------
   # 5. Cấu hình /etc/fstab (tự động mount khi boot)
   # -----------------------------------------------------------
   # Mẫu /etc/fstab cho Banana Pi boot từ SD card:
   #
   # <device>       <mount>   <type>  <options>          <dump> <pass>
   /dev/mmcblk0p1  /boot     vfat    defaults            0      2
   /dev/mmcblk0p2  /         ext4    defaults,noatime    0      1
   tmpfs           /tmp      tmpfs   defaults,size=64M   0      0
   tmpfs           /var/log  tmpfs   defaults,size=16M   0      0

   # Dùng UUID thay vì device name (bền vững hơn)
   # Lấy UUID:
   sudo blkid /dev/mmcblk0p2
   # Output: /dev/mmcblk0p2: UUID="a1b2c3d4-..." TYPE="ext4"

   # /etc/fstab với UUID:
   # UUID=a1b2c3d4-... / ext4 defaults,noatime 0 1

   # -----------------------------------------------------------
   # 6. Kiểm tra & sửa lỗi filesystem
   # -----------------------------------------------------------
   # Kiểm tra lỗi (unmount trước khi chạy)
   sudo umount /dev/mmcblk0p2
   sudo fsck.ext4 -f /dev/mmcblk0p2

   # Kiểm tra nhanh
   sudo fsck -N /dev/mmcblk0p2  # Chỉ xem, không sửa
   sudo fsck -y /dev/mmcblk0p2  # Tự động trả lời yes

   # -----------------------------------------------------------
   # 7. Script tự động format và mount (cho SD card mới)
   # -----------------------------------------------------------
   # format_sd.sh - Chạy trên host PC
   # #!/bin/bash
   # DEV=/dev/mmcblk0
   #
   # echo "=== Xoá partition cũ ==="
   # sudo dd if=/dev/zero of=$DEV bs=1M count=10
   #
   # echo "=== Tạo partition ==="
   # sudo fdisk $DEV <<EOF
   # n
   # p
   # 1
   # 
   # +64M
   # t
   # b
   # n
   # p
   # 2
   # 
   # 
   # w
   # EOF
   #
   # echo "=== Format ==="
   # sudo mkfs.vfat -F 32 -n BOOT ${DEV}p1
   # sudo mkfs.ext4 -L rootfs ${DEV}p2
   #
   # echo "=== Mount ==="
   # sudo mount ${DEV}p2 /mnt
   # sudo mkdir -p /mnt/boot
   # sudo mount ${DEV}p1 /mnt/boot
   #
   # echo "Done! Rootfs mounted at /mnt, boot at /mnt/boot"

---

.. rubric:: 11. Debug & Phân tích

.. code-block:: bash

   # Theo dõi system call / library call
   strace <cmd>
   ltrace <cmd>

   # Xem danh sách file đang mở
   lsof

   # Phân tích file nhị phân
   file <bin>                  # Xác định loại file
   ldd <bin>                   # Xem thư viện liên kết
   readelf -a <bin>            # Xem thông tin ELF
   objdump -d <bin>            # Disassemble

---

.. rubric:: Tài liệu tham khảo

- `Linux man pages <https://man7.org/linux/man-pages/>`_
- `Ubuntu man pages <https://manpages.ubuntu.com/>`_
