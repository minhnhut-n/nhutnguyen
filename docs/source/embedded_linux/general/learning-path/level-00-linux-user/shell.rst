Shell
=====

- ✅ Bash
- ✅ SSH
- ✅ tmux/screen
- ✅ vim/nano
- ✅ systemd
- ✅ cron

1. Bash
-------

Bash (Bourne Again SHell) là một CLI giúp tự động hóa tuần tự các công việc thông qua
một chuỗi các đoạn mã thực thi mà không cần biên dịch trước. Bash chuyển tuần tự các
code để chạy (trình thông dịch - chuyển trực tiếp các mã nguồn code sang ngôn ngữ máy
để chạy thay vì phải biên dịch trước).

Chi tiết hướng dẫn và học về ngôn ngữ bash có thể tham khảo tại:

- https://www.w3schools.com/bash/bash_getstarted.php

Mẹo: Bash chỉ là một kỹ năng phụ trong quá trình học, không cần quá đầu tư ngay từ đầu,
dùng tới đâu học tới đó.

2. SSH (Secure Shell)
---------------------

SSH là giao thức mạng cho phép truyền dữ liệu không dây một cách bảo mật.

Lưu ý: Các kết nối shell thường thấy là ở local (cho test). Để sử dụng shell cho remote
control thì bạn phải thông qua các phần mềm hoặc công cụ port-forwarding như Tailscale
hay WireGuard.

Một số lệnh SSH hữu ích:

.. code-block:: bash

   # Generate keypair, file sẽ được lưu vào ~/.ssh/
   $ ssh-keygen

   # Login/remote vào bất kỳ Linux server/machine nào có SSH
   $ ssh <username>@<ip_addr>
   # Sau đó nhập password

   # File transmission qua SSH - Upload (local -> remote)
   scp <path_of_src> <username>@<ip_addr>:<path_of_destination>

   # Download (remote -> local)
   scp <username>@<ip_addr>:<path_of_source_remote> <path_of_destination>

   # Với folder, thêm flag -r ngay sau lệnh scp
   scp -r <path_of_src> <username>@<ip_addr>:<path_of_destination>

Chi tiết options có thể xem với ``--help``.

3. tmux / screen
----------------

Do nhu cầu của người dùng Linux thường không có GUI, CLI là cách chính để giao tiếp
giữa người dùng và máy tính. Tmux giải quyết vấn đề multi-task, cho phép mở nhiều
cửa sổ terminal trên cùng một terminal app để chạy song song (parallel task).

"Screen" là công cụ được tích hợp sẵn vào trong linux, nhưng lại ít được cập nhật thậm
chí còn bị lược bỏ trong các bản phát hành linux, còn "tmux" hiện đại hơn và có thể dể
dàng cái đặt thông qua lệnh.

Tmux sẽ mở lại chính session nếu nó bị ngắt kết nối mạng giữa chừng thay vì phải mở và cấu
hình lại. Điểm khác nhau giữa tmux và mobaxterm (một công cụ hỗ trợ ssh) là khi tab moba đóng
các session sẽ bị kill và các service đang hoạt động cũng vậy, nhưng với tmux các tiến trình
vẫn chạy bình thường.

Ứng dụng này đặc biệt quan trọng trong các trường hợp sử dụng là: máy chủ, build code c/c++,
train AI, script run,... Chỉ cần mở mobaxterm hoặc putty từ bất kì máy nào và gõ "tmux attach"
trạng thái làm việc sẽ được khôi phục.

.. code-block:: bash

   # Install tmux
   sudo apt-get update
   sudo apt-get install tmux

   # New session with name
   tmux new -s s_name

   # Or basic just new
   tmux

   # List all session
   tmux ls

   # Detach session (Ctrl+b d)
   # Attach lại session
   tmux attach -t s_name

   # Split pane ngang (Ctrl+b ")
   # Split pane dọc (Ctrl+b %)
   # Di chuyển giữa các pane (Ctrl+b + arrow keys)

   # Thoát tmux session
   ctrl + b, d

   # Xóa một tmux session
   tmux kill-session -t <session_name>

   # Xóa toàn bộ session
   tmux kill-server

4. vim / nano
-------------

Hai trình soạn thảo văn bản phổ biến trong terminal.
Nano phù hợp cho người mới bắt đầu với các thao tác đơn giản:

.. code-block:: bash

   # Mở file với nano
   nano filename.txt

   # Thoát: Ctrl+X
   # Lưu: Ctrl+O
   # Tìm kiếm: Ctrl+W

Vim mạnh mẽ hơn nhưng có learning curve cao hơn:

.. code-block:: bash

   # Mở file với vim
   vim filename.txt

   # Các chế độ trong vim:
   # - Normal mode: mode mặc định khi mở vim
   # - Insert mode:  nhấn i để vào chế độ soạn thảo
   # - Visual mode: nhấn v để chọn văn bản
   # - Command mode: nhấn : để gõ lệnh

   # Lệnh cơ bản:
   # :q  - thoát
   # :w  - lưu
   # :wq - lưu và thoát
   # :q! - thoát không lưu

5. systemd
----------

Systemd là một bộ các công cụ cơ bản để xây dụng các khối (blocks) cho một hệ thống linux.
Nó cung cấp một hệ thống và là một bộ điều khiển hoạt động của các service khác, với PID là 1
và chạy phần còn lại của cả hệ thống.

Systemd (deamon) này tồn tại cơ chế mạnh mẽ có khả năng chạy song song hiệu quả (aggressive
parallelization capabilities) - xin thứ lỗi tại hạ tiếng anh lởm, thông qua socket và D-Bus
để:

- kích hoạt các services. (service activation).
- theo dõi các process thông qua cơ chế cgroup.
- duy trì các mount point (điểm ghép nối) và auto mount point.
- service control logic (chịu trách nhiệm quản lý vòng đời của một service)

"implements an elaborate transactional dependency-based service control logic."

Service control logic (Lập trình / Logic điều khiển dịch vụ):
Là phần code chịu trách nhiệm quản lý vòng đời của các dịch vụ (bắt đầu, dừng, khởi động lại, kiểm tra trạng thái...).

Dependency-based (Dựa trên sự phụ thuộc):
Dịch vụ này phụ thuộc vào dịch vụ khác để chạy.
Ví dụ: Dịch vụ Web Server chỉ được phép chạy sau khi dịch vụ Database đã khởi động xong.

Transactional (Có tính giao dịch / Giao dịch nguyên tố - Atomicity):
Tuân theo nguyên tắc "được ăn cả, ngã về không". Nếu trong quá trình khởi động hoặc dừng một chuỗi dịch vụ mà có một dịch vụ bị lỗi,
hệ thống sẽ tự động Rollback (hoàn tác) toàn bộ về trạng thái an toàn trước đó.

Elaborate (Phức tạp / Tinh vi / Được thiết kế tỉ mỉ):
Mô tả sự phức tạp của logic, xử lý nhiều trường hợp biên (edge cases), ngoại lệ và luồng điều khiển nâng cao.

Reference: https://systemd.io/

Các hạng mục trong systemd là:

Nhóm 1: Quản lý dịch vụ hệ thống
  - systemctl, quản lý dịch vụ, trong đó có start/stop/restart/enable/disable, điều khiển nguồn reboot/poweroff.
  - systemd-analyze, hiệu năng khởi động của hệ thống (booting time)

Nhóm 2: Nhật ký (Logs)
  - journalctl, xem và lọc các nhật ký hệ thống, nhân kernel, hoặc các khoảng thời gian cụ thể.
  - systemd-cat, chuyển hướng đầu ra của một lệnh bất kì hoặc một file text vào hệ thống của journald.

Nhóm 3: System configuration
  - hostnamectl, thay đổi tên máy tính, thông tin kiến trúc hệ điều hành.
  - timedatectl, quản lý thời gian, múi giờ, đồng bộ thời gian qua mạng NTP.
  - localectl, cấu hình ngôn ngữ hệ thống và sơ đồ bàn phím (keyboard layout).

Nhóm 4: Sandbox
  - coredumpctl, tìm kiếm và phân tích các file coredump(dữ liệu lưu lại khi một chương trình crash/lỗi).
  - machinectl, quản lý và tương tác với các container hoặc máy ảo chạy bằng nspawn.

.. code-block:: bash

   # Kiểm tra trạng thái service
   $ systemctl status <service_name>

   # Tắt service
   $ systemctl stop <service>

   # Reboot service (ví dụ khi zombie task)
   $ systemctl restart <service>

   # Enable boot up with system (khi cold boot)
   $ systemctl enable <service>

   # Xem log của service
   $ journalctl -u <service_name>

6. Cron
-------

Cron là một deamon chạy ngầm, có trách nhiệm dọn rác và quét dọn kĩ càng các tệp tin hệ thống,
chịu trách nhiệm tự động hóa các công việc có tính chất chu kì, ví dụ như backup database, dọn
rác trong thư mục.

Tất cả chỉ cần làm là đặt vào cron tab /opt/... một hoặc vài đoạn script, nó sẽ được trigger mỗi
phút và thực hiện các tác vụ như yêu cầu, tránh phải thực hiện một hành động lặp lại tốn thời gian.

.. code-block:: bash

   # Edit crontab cho user hiện tại
   $ crontab -e

   # List crontab hiện tại
   $ crontab -l

   # Cấu trúc cron:
   # * * * * * command
   # - - - - -
   # | | | | |
   # | | | | +---- Day of week (0-7, 0=Sun)
   # | | | +------ Month (1-12)
   # | | +-------- Day of month (1-31)
   # | +---------- Hour (0-23)
   # +------------ Minute (0-59)

   # Ví dụ: Chạy script mỗi ngày lúc 2:30 sáng
   30 2 * * * /path/to/script.sh

   # ==========================================
   # 1. KHỞI ĐỘNG VÀ KÍCH HOẠT (START & ENABLE)
   # ==========================================

   # Khởi động dịch vụ ngay lập tức
   sudo systemctl start cron

   # Bật tự động khởi động cùng hệ điều hành (khi reboot)
   sudo systemctl enable cron

   # LỆNH GỘP: Vừa bật tự khởi động, vừa khởi động ngay lập tức
   sudo systemctl enable --now cron

   # ==========================================
   # 2. KIỂM TRA TRẠNG THÁI (STATUS & LOGS)
   # ==========================================

   # Xem trạng thái chi tiết (đang chạy hay đã dừng, xem log gần nhất)
   sudo systemctl status cron

   # Kiểm tra nhanh dịch vụ có đang hoạt động hay không (active/inactive)
   sudo systemctl is-active cron

   # Kiểm tra xem dịch vụ đã bật tự khởi động cùng máy chưa (enabled/disabled)
   sudo systemctl is-enabled cron

   # Kiểm tra tiến trình cron có thực sự chạy trên RAM hay không
   ps aux | grep cron

   # ==========================================
   # 3. LÀM MỚI VÀ KHỞI ĐỘNG LẠI (RELOAD & RESTART)
   # ==========================================

   # Tải lại cấu hình (áp dụng file cấu hình mới mà không làm gián đoạn tác vụ đang chạy)
   sudo systemctl reload cron

   # Khởi động lại toàn bộ dịch vụ (Tắt đi rồi Bật lại)
   sudo systemctl restart cron

   # ==========================================
   # 4. DỪNG VÀ VÔ HIỆU HÓA (STOP & DISABLE)
   # ==========================================

   # Dừng dịch vụ ngay lập tức (các lịch trình sẽ không chạy nữa)
   sudo systemctl stop cron

   # Tắt tự động khởi động cùng hệ điều hành (reboot sẽ không tự bật)
   sudo systemctl disable cron

   # LỆNH GỘP: Vừa dừng dịch vụ, vừa tắt tự khởi động cùng hệ điều hành
   sudo systemctl disable --now cron

   # Khóa hoàn toàn dịch vụ (ngăn tất cả các lệnh khác vô tình start nó lên)
   sudo systemctl mask cron

   # Mở khóa dịch vụ (nếu trước đó đã dùng lệnh mask)
   sudo systemctl unmask cron

   # ------------------------------------------
   # *Lưu ý: Nếu dùng hệ điều hành dòng RedHat/CentOS/RHEL/Fedora,
   # hãy thay thế chữ "cron" ở cuối mỗi lệnh thành "crond".
   # ------------------------------------------

Các lỗi thường gặp với Cron là:

- Lỗi môi trường, vì là task chạy ngầm nên trong nhiều trường hợp nên viết dưới dạng đường dẫn tuyệt đối.
- Chưa cấp quyền thực thi cho file.
- Không kiểm tra log khi lỗi, bằng các câu lệnh:

  - journal -u cron
  - tail -f /var/log/syslog | grep cron

Tham khảo thêm: https://viblo.asia/p/cron-job-la-gi-huong-dan-su-dung-cron-tab-E375zLo2ZGW
