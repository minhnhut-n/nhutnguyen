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

.. rubric:: 10. Debug & Phân tích

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
