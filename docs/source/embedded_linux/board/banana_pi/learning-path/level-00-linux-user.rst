Level 0 – Linux User (Beginner)
================================

Mục tiêu: Thành thạo Linux như môi trường làm việc.

Hướng dẫn này dành cho người mới bắt đầu làm quen với Linux. Bạn sẽ học cách sử dụng
shell, quản lý filesystem, các command line tools cơ bản và hiểu về process trong Linux.

Shell
-----

- ✅ Bash
- ✅ SSH
- ✅ tmux/screen
- ✅ vim/nano
- ✅ systemd
- ✅ cron

**1. Bash**

Bash (Bourne Again SHell) là một CLI giúp tự động hóa tuần tự các công việc thông qua
một chuỗi các đoạn mã thực thi mà không cần biên dịch trước. Bash chuyển tuần tự các
code để chạy (trình thông dịch - chuyển trực tiếp các mã nguồn code sang ngôn ngữ máy
để chạy thay vì phải biên dịch trước).

Chi tiết hướng dẫn và học về ngôn ngữ bash có thể tham khảo tại:

- https://www.w3schools.com/bash/bash_getstarted.php

Mẹo: Bash chỉ là một kỹ năng phụ trong quá trình học, không cần quá đầu tư ngay từ đầu,
dùng tới đâu học tới đó.

**2. SSH (Secure Shell)**

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

**3. tmux / screen**

Do nhu cầu của người dùng Linux thường không có GUI, CLI là cách chính để giao tiếp
giữa người dùng và máy tính. Tmux giải quyết vấn đề multi-task, cho phép mở nhiều
cửa sổ terminal trên cùng một terminal app để chạy song song (parallel task).

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

**4. vim / nano**

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

**5. systemd**

Systemd là "chúa của các service" - service đầu tiên khi kernel boot lên. Nó là main
core để chạy và watchdog các service trong Linux system. Trong các sản phẩm nhúng như
TV hoặc các thiết bị khác, systemd được dùng để backup hoặc recover các service khi
có crash hoặc dump.

Còn tại sao lại là back-up hả? Do là "lào gì cũng tôn" thì chậm chứ sao. Nên thường
như các product của "chê bôn company" họ có một cái service để guide riêng các flow 
cho software chạy.

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

**6. Cron**

Cron là chương trình để xử lý các tác vụ lặp đi lặp lại. Cron Job đưa ra một lệnh để
lên lịch "làm việc" cho một hành động cụ thể, tại một thời điểm cụ thể mà cần lặp đi
lặp lại.

Ví dụ: Ứng dụng có chức năng lưu tạm file, sau 3 ngày cần dọn các file tạm đó đi.
Đối với các công việc định kỳ, lặp đi lặp lại thì cron là giải pháp hoàn hảo.

Cron là một daemon (chạy dưới nền) để thực thi những tác vụ không cần tương tác.

File crontab mặc định: ``/etc/crontab`` và nằm trong thư mục ``/etc/cron.*/``.

(riêng mỗi cái phần này là mình nhờ AI viết nhé, các bạn đọc cái link bên dưới cho trực
quan dể hiểu, mình buồn ngủ quá rồi.)

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

Tham khảo thêm: https://viblo.asia/p/cron-job-la-gi-huong-dan-su-dung-cron-tab-E375zLo2ZGW

Filesystem
----------

- ext4
- procfs
- sysfs
- devtmpfs
- tmpfs

**ext4**

Là filesystem mặc định phổ biến nhất trên Linux, hỗ trợ dung lượng lớn lên đến 1 exabyte,
kích thước file tối đa 16 TB (với block size 4KB). ext4 là phiên bản kế thừa của ext3
với các cải tiến về hiệu suất, hỗ trợ journaling (phục hồi dữ liệu khi crash) và
extents (giảm fragmentation).

.. code-block:: bash

   # Xem thông tin filesystem
   $ df -hT

   # Kiểm tra dung lượng ổ đĩa
   $ df -h

   # Kiểm tra dung lượng thư mục
   $ du -sh /path/to/dir

**procfs**

Là filesystem ảo (pseudo filesystem) mount tại ``/proc``, cung cấp thông tin về
kernel và processes đang chạy. Mỗi process có một thư mục riêng với PID tương ứng
bên trong ``/proc``.

.. code-block:: bash

   # Xem thông tin CPU
   $ cat /proc/cpuinfo

   # Xem thông tin memory
   $ cat /proc/meminfo

   # Xem thông tin process cụ thể (PID=1 là init/systemd)
   $ cat /proc/1/status

**sysfs**

Là filesystem ảo mount tại ``/sys``, cung cấp thông tin về hardware devices,
drivers và kernel objects. sysfs xuất hiện từ kernel 2.6 để thay thế cho các
file rác rưởi trong procfs.

.. code-block:: bash

   # Xem danh sách block devices
   $ ls /sys/block/

   # Xem thông tin về một device cụ thể (vd: sda)
   $ cat /sys/block/sda/size

   # Xem class devices
   $ ls /sys/class/

**devtmpfs**

Là filesystem ảo mount tại ``/dev``, tự động tạo các device nodes cho hardware
được kernel phát hiện. Trước đây, việc tạo device nodes phải làm thủ công với
``mknod``, devtmpfs tự động hóa quá trình này.

.. code-block:: bash

   # Xem các device nodes
   $ ls /dev/

   # Một số device nodes phổ biến:
   # /dev/sda   - ổ cứng SCSI/SATA đầu tiên
   # /dev/ttyUSB0 - USB-to-Serial adapter
   # /dev/mmcblk0 - thẻ nhớ SD/MMC
   # /dev/null  - "black hole" device

**tmpfs**

Là filesystem ảo lưu trữ dữ liệu trong RAM (volatile memory). Dữ liệu trên tmpfs
sẽ mất khi reboot. Thường được dùng cho các file tạm thời, cache, hoặc shared memory.

.. code-block:: bash

   # Xem các tmpfs đang mounted
   $ df -hT | grep tmpfs

   # Mount tmpfs thủ công
   $ sudo mount -t tmpfs -o size=100M tmpfs /mnt/mytmp

Command line
------------

- ps
- top
- htop
- free
- vmstat
- iostat
- netstat
- ss
- lsof

**ps** - Process Status: Hiển thị danh sách processes đang chạy.

.. code-block:: bash

   $ ps aux                    # Xem tất cả processes
   $ ps -ef                    # Format khác
   $ ps -u username            # Processes của user cụ thể

**top** - Real-time process monitoring: Xem processes theo thời gian thực.

.. code-block:: bash

   $ top                       # Mở giao diện real-time
   # Trong top: nhấn 'q' để thoát, 'k' để kill process

**htop** - Enhanced version of top (cần cài đặt riêng): Giao diện đẹp hơn, hỗ trợ
mouse, dễ sử dụng hơn top.

.. code-block:: bash

   # Install htop
   sudo apt-get install htop
   $ htop

**free** - Xem dung lượng RAM và swap.

.. code-block:: bash

   $ free -h                   # Hiển thị với đơn vị GB/MB
   $ free -m                   # Hiển thị với đơn vị MB

**vmstat** - Virtual Memory Statistics: Báo cáo về memory, processes, CPU, I/O.

.. code-block:: bash

   $ vmstat 2                  # Cập nhật mỗi 2 giây
   $ vmstat -s                 # Thống kê tổng hợp

**iostat** - Input/Output Statistics: Báo cáo về CPU và I/O của devices.

.. code-block:: bash

   # Install sysstat package
   sudo apt-get install sysstat
   $ iostat -x 2               # Extended stats, mỗi 2 giây

**netstat** - Network Statistics: Xem các kết nối mạng, routing tables, interface stats.

.. code-block:: bash

   $ netstat -tulpn            # Xem các ports đang listen
   $ netstat -an               # Tất cả kết nối

**ss** - Socket Statistics (modern replacement for netstat): Nhanh hơn và thông tin
chi tiết hơn netstat.

.. code-block:: bash

   $ ss -tulpn                 # Xem các ports đang listen
   $ ss -s                     # Tổng quan socket stats

**lsof** - List Open Files: Liệt kê tất cả các file đang được mở bởi processes.

.. code-block:: bash

   $ lsof -i :80               # Process nào đang dùng port 80
   $ lsof -u username          # File mở bởi user
   $ lsof /path/to/file        # Process nào đang dùng file này

Process
-------

- fork
- exec
- signal
- pipe

**fork**

Là system call tạo ra một process mới (child process) bằng cách copy chính xác
process hiện tại (parent process). Child process nhận một bản sao của address space,
file descriptors, environment variables,... của parent.

.. code-block:: bash

   # Xem process tree
   $ pstree

   # Xem parent PID (PPID) của process
   $ ps -o pid,ppid,cmd

**exec**

Là system call thay thế process image hiện tại bằng một program mới. Khi một
process gọi exec, nó sẽ load program mới vào memory và bắt đầu chạy program đó,
PID của process vẫn giữ nguyên nhưng code, data, stack đều được thay thế.

Sự kết hợp fork + exec là cách Linux tạo ra process mới chạy program khác:

1. fork() tạo child process
2. exec() thay thế child process bằng program mới

**signal**

Là cơ chế giao tiếp bất đồng bộ (asynchronous) giữa các processes hoặc giữa kernel
và process. Signal thông báo cho process về một sự kiện đặc biệt.

.. code-block:: bash

   # Gửi signal đến process
   $ kill -SIGTERM <PID>       # Yêu cầu process kết thúc (graceful)
   $ kill -SIGKILL <PID>       # Buộc process kết thúc ngay lập tức
   $ kill -SIGUSR1 <PID>       # User-defined signal 1

   # Một số signal phổ biến: (copy no righter :)) 
   # SIGINT (2)  - Ctrl+C, interrupt từ bàn phím
   # SIGTERM (15)- Yêu cầu kết thúc
   # SIGKILL (9) - Buộc kết thúc
   # SIGSTOP (19)- Tạm dừng process
   # SIGCONT (18)- Tiếp tục process bị tạm dừng

**pipe**

Là cơ chế IPC (Inter-Process Communication) cho phép output của process này trở
thành input của process khác. Pipe được ký hiệu bằng ``|`` trong shell.

.. code-block:: bash

   # Ví dụ: Tìm process đang chạy với tên "nginx"
   $ ps aux | grep nginx

   # Ví dụ: Đếm số file trong thư mục
   $ ls -l | wc -l

   # Anonymous pipe (shell pipe): chỉ dùng giữa parent-child processes
   # Named pipe (FIFO): có thể dùng giữa các processes không liên quan
   $ mkfifo mypipe            # Tạo named pipe
   $ echo "hello" > mypipe    # Ghi vào pipe (blocking)
   $ cat mypipe               # Đọc từ pipe