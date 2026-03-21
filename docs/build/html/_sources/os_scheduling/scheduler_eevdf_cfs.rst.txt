Scheduler in OS: EEVDF and CFS
==============================

Tác giả: Nguyễn Minh Nhựt

Giới thiệu
----------

CFS đã xuất hiện từ Linux kernel 2.6 nhằm giải quyết bài toán phân bổ runtime và tài nguyên một cách công bằng. Thông qua các cơ chế phổ biến như `cgroup`, CFS cũng hỗ trợ quy hoạch (scheduling) đơn lẻ.

EEVDF (Earliest Eligible Virtual Deadline) là một bộ lập lịch mới được đầu tư mạnh từ năm 2023. Có thể xem EEVDF như “CFS+”, bổ sung cơ chế kiểm soát deadline của process theo thời gian thực.

Việc hiểu cả hai cơ chế giúp ta so sánh được các khác biệt cốt lõi và lý do EEVDF ra đời.

Mục lục nội dung
---------------

Các mục chính trong tài liệu:

- Memory space
- PCB - Process Control Block & Process State
- CFS
- EEVDF
- So sánh nhanh

Memory space
------------

Không gian bộ nhớ thường có các segment:

- Chương trình (code): đọc trực tiếp từ bộ nhớ tĩnh (non-volatile storage).
- Data: biến toàn cục và static vars, cấp phát và init trước khi chương trình chạy.
- Heap: bộ nhớ động, phục vụ `malloc`, `calloc`, `alloc`, `delete`, `free`.
- Stack: lưu biến cục bộ (local vars) trong phạm vi scope; ra ngoài scope thì biến không còn hiệu lực.

PCB - Process Control Block & Process State
-------------------------------------------

Process control block (PCB) lưu các thông tin quan trọng của process:

- Identifier / PID.
- Trạng thái process: running, ready, halted, completed.
- Thông tin lập lịch: process được xếp trong queue nào.
- Thanh ghi CPU / program counter trong quá trình swap.
- Trạng thái I/O.
- Memory page của process.
- Các thông tin quản trị khác.

Process state (vòng đời cơ bản):

- Null -> Start: tiến trình được khởi tạo.
- Start -> Ready: vào hàng chờ.
- Ready -> Running: được cấp CPU để chạy.
- Running -> Terminate: bị kill/suspend hoặc finished.
- Running -> Wait: tạm dừng, chờ sự kiện.
- Wait -> Ready: được trigger và quay lại hàng chờ.

CFS Scheduling
--------------

Khái niệm nền tảng:

- Timeslice: lát cắt thời gian giữa các lần thực thi (time interval/quanta).
- Priority: mức độ ưu tiên (int). Trong Linux, giá trị càng lớn thì ưu tiên càng thấp.
- Nice value: số nguyên điều chỉnh nhẹ giữa các task cùng mức ưu tiên.
- Virtual runtime: thời gian ảo dùng để đánh giá công bằng, đối chiếu với real time.

Timeslice được phân bổ dựa trên 3 yếu tố:

- Priority (static) do người dùng đặt (dynamic do CFS tính).
- Số lượng task trong run queue hiện tại.
- Tải (load) của task.

Công thức timeslice:

Timeslice = period * (task_load / cfs_load)

Giải thích biến:

- Task load: giá trị timeslice trước đó mà task được phân bổ.
- CFS load: trọng số do CFS scheduler định nghĩa.
- Period: load của run-queue hiện tại.

Phân loại task trong Linux kernel scheduler:

- Realtime task (RT).
- Batch task (normal task).
- Interactive task (tương tác trực tiếp với user).

Policy lập lịch (scheduling policy):

Linux sử dụng một số policy cố định: RR, FIFO, OTHER, IDLE, BATCH. Thứ tự và quyền ưu tiên phụ thuộc vào:

- `sched_policy`: RR / FIFO / OTHER / ...
- `sched_priority`: dùng để set priority cho từng policy.

Giá trị static:

- PRIORITY: 0 -> 99 cho RT task.
- PRIORITY: 100 -> 139 cho các task còn lại.
- NICE: -20 -> +10.

Giá trị dynamic (CFS) được tính từ static value và scheduler boost. Hai yếu tố ảnh hưởng chính:

- CPU time.
- I/O operation.

Cơ chế hoạt động (tóm tắt):

User config (nice value + priority) -> CFS calculation & timeslices -> insert vào cây nhị phân (RB-tree) -> run queue -> time out -> lặp lại.

Lưu ý: các bước insert/sort/run queue là những quy trình riêng, CFS chủ yếu tập trung vào tính công bằng theo virtual runtime. Task có virtual runtime nhỏ nhất được chạy trước (đỉnh của cây nhị phân).

.. note::
   Tài liệu gốc có nhắc đến một bảng phân loại và một timeline minh họa CFS nhưng chưa đính kèm. Có thể bổ sung sau khi có hình/diagram.

EEVDF (Earliest Eligible Virtual Deadline)
------------------------------------------

EEVDF tập trung lập lịch cho các deadline task để đảm bảo hoàn thành đúng hạn. Cơ chế này bổ sung thêm khái niệm về LAG và kiểm soát thời gian thực.

Hạn chế của CFS (động lực ra đời EEVDF):

- RB-tree sắp xếp theo virtual runtime nên task interactive có thể bị delay.
- Task “nice” cao (ưu tiên thấp) có thể bị đói CPU.
- Không đảm bảo số timeslice phù hợp với task cần deadline.

LAG và eligibility:

LAG = Virtual runtime - Actual runtime

- Nếu LAG > 0: task đã “giữ CPU đủ / quá mức”.
- Nếu LAG <= 0: task bị đói CPU.

Task chỉ eligible khi thuộc trường hợp LAG <= 0. Actual runtime được đo bằng real time, vì vậy việc lập lịch nghiêm túc hơn và giảm được vấn đề chậm trễ cho task tương tác.

Virtual Deadline:

VD (Virtual Deadline) = Eligible + (Timeslice / Weight)

Task quan trọng (weight cao) sẽ có deadline nhỏ hơn và được ưu tiên chạy sớm hơn.

So sánh nhanh
-------------

- CFS ưu tiên tính công bằng dựa trên virtual runtime.
- EEVDF ưu tiên task đủ điều kiện (eligible) và có deadline sớm nhất.
- EEVDF cải thiện tình trạng starvation và độ trễ của interactive task.

Lệnh hữu ích để kiểm tra
-------------------------

Tìm `nice` và static value của process:

.. code-block:: console

   ps -p <pid> -o pid,ni,comm
   cat /proc/<pid>/stat | awk '{print "Static:", $18, "Nice:", $19}'

Tìm thông tin process:

.. code-block:: console

   ps ax | grep <process name/service name>

Mở rộng terminal và xem task manager:

.. code-block:: console

   COLUMN=200 top

Xem các properties của scheduler:

.. code-block:: console

   cat /proc/sys/kernel/sched*
   sysctl -a | grep sched
