Setup
=====

Hướng dẫn cài đặt và cấu hình Banana Pi M4.

.. rubric:: Yêu cầu phần cứng

- Banana Pi M4
- MicroSD card (16GB trở lên)
- Nguồn 5V/3A
- Cáp HDMI
- Bàn phím và chuột

---

.. rubric:: Cài đặt hệ điều hành

**Tải image Ubuntu Server:**

.. code-block:: bash

   wget https://example.com/ubuntu-server-arm64.img

**Flash image vào MicroSD:**

.. code-block:: bash

   sudo dd if=ubuntu-server-arm64.img of=/dev/sdX bs=4M status=progress

---

.. rubric:: Cấu hình ban đầu

**Kết nối và đăng nhập:**

.. code-block:: bash

   ssh user@banana-pi-m4.local
   # Password: bananapi

**Cập nhật hệ thống:**

.. code-block:: bash

   sudo apt update
   sudo apt upgrade -y

**Cài đặt các gói cần thiết:**

.. code-block:: bash

   sudo apt install -y vim git build-essential

---

.. rubric:: Kiểm tra thông tin hệ thống

.. code-block:: bash

   # Xem thông tin CPU
   cat /proc/cpuinfo

   # Xem thông tin bộ nhớ
   free -h

   # Xem thông tin board
   cat /proc/device-tree/model

---

.. rubric:: Tài liệu tham khảo

- `Banana Pi Official Website <https://www.banana-pi.org/>`_
- `Banana Pi M4 Manual <https://wiki.banana-pi.org/Banana_Pi_BPI-M4>`_