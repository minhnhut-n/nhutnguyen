Level 1 – Linux System Programming
===================================

Ngôn ngữ C là ngôn ngữ chính cho toàn bộ kernel module, device driver và các ứng dụng hệ thống - không có ngoại lệ. Kiến thức cơ bản về lập trình C là cần thiết để phát triển các ứng dụng Linux nhúng, vì C cho phép tương tác trực tiếp với phần cứng và đơn giản nhất để con người đọc và hiểu được.

Có bạn hỏi tại sao lại không phải là C++? Vì C++ là ngôn ngữ hướng đối tượng, trong khi C lại nghiêng về thiết kế và tối ưu hệ thống. C++ có thể có nhiều class "rác" không tận dụng 100% tài nguyên, hoặc do tính chất của OOP được dùng làm future feature.

Đồng ý rằng code C trong kernel phải được thiết kế theo nguyên tắc tường minh, dễ đọc, dễ mở rộng và tối ưu.
---

Các system call cốt lõi để tương tác với kernel Linux.

**open()** - Mở file hoặc device.

.. code-block:: c

   #include <fcntl.h>
   #include <unistd.h>

   int fd = open("/path/to/file", O_RDWR | O_CREAT, 0644);
   if (fd < 0) {
       perror("open failed");
       return -1;
   }

Các flag thường dùng: ``O_RDONLY``, ``O_WRONLY``, ``O_RDWR``, ``O_CREAT``,
``O_TRUNC``, ``O_APPEND``, ``O_NONBLOCK``.

**close()** - Đóng file descriptor.

.. code-block:: c

   close(fd);

Luôn kiểm tra và đóng fd sau khi sử dụng để tránh rò rỉ tài nguyên.

**read()** - Đọc dữ liệu từ file descriptor.

.. code-block:: c

   #include <unistd.h>

   char buf[1024];
   ssize_t n = read(fd, buf, sizeof(buf));
   if (n < 0) {
       perror("read failed");
   }

Trả về số byte đã đọc, 0 nếu EOF, -1 nếu lỗi.

**write()** - Ghi dữ liệu vào file descriptor.

.. code-block:: c

   const char *msg = "Hello, Linux!";
   ssize_t n = write(fd, msg, strlen(msg));
   if (n < 0) {
       perror("write failed");
   }

**ioctl()** - Input/Output Control: Thao tác với thiết bị phần cứng hoặc
các tham số đặc biệt của device driver.

.. code-block:: c

   #include <sys/ioctl.h>

   int baud = B115200;
   ioctl(fd, TIOCSSERIAL, &baud);

Dùng phổ biến trong UART, GPIO, SPI, I2C, network interfaces.

**mmap()** - Memory-Map File: Ánh xạ file hoặc device vào bộ nhớ process
để truy cập trực tiếp như một mảng.

.. code-block:: c

   #include <sys/mman.h>

   void *addr = mmap(NULL, length, PROT_READ | PROT_WRITE,
                     MAP_SHARED, fd, 0);
   if (addr == MAP_FAILED) {
       perror("mmap failed");
   }

   // Truy cập dữ liệu trực tiếp qua addr
   printf("%s", (char *)addr);

   munmap(addr, length);  // Giải phóng khi không dùng nữa

**poll()** - I/O Multiplexing: Theo dõi nhiều file descriptors cùng lúc
để kiểm tra sự kiện đọc/ghi/lỗi.

.. code-block:: c

   #include <poll.h>

   struct pollfd fds[2];
   fds[0].fd = fd1; fds[0].events = POLLIN;
   fds[1].fd = fd2; fds[1].events = POLLIN;

   int ret = poll(fds, 2, 5000);  // timeout 5 giây
   if (ret > 0) {
       if (fds[0].revents & POLLIN) {
           // fd1 có dữ liệu để đọc
       }
   }

**epoll()** - I/O Event Notification (Linux-specific): Phiên bản cải tiến
của poll, hiệu quả hơn với hàng ngàn file descriptors.

.. code-block:: c

   #include <sys/epoll.h>

   int epfd = epoll_create1(0);
   struct epoll_event ev;
   ev.events = EPOLLIN;
   ev.data.fd = fd;
   epoll_ctl(epfd, EPOLL_CTL_ADD, fd, &ev);

   struct epoll_event events[10];
   int n = epoll_wait(epfd, events, 10, -1);
   for (int i = 0; i < n; i++) {
       if (events[i].events & EPOLLIN) {
           // Xử lý dữ liệu từ events[i].data.fd
       }
   }

   close(epfd);

IPC (Inter-Process Communication)
----------------------------------

- pipe
- shared memory
- semaphore
- message queue

IPC là tập hợp các phương thức truyền tải dữ liệu giữa các process trên
Linux system, trong đó các services/process có thể gửi dữ liệu trực tiếp
cho nhau thông qua các kênh truyền dẫn.

Có hai cách chính để chia sẻ thông tin dựa trên IPC:

- Gửi tin nhắn (message) thông qua PIPE
- Thiết lập memory sharing (dùng chung)

**pipe**

Giao thức đường ống, tưởng tượng như dòng chảy dầu khí/nước.

.. code-block:: c

   #include <unistd.h>

   int pipefd[2];
   pipe(pipefd);  // pipefd[0] = read end, pipefd[1] = write end

   if (fork() == 0) {
       // Child process: đọc từ pipe
       close(pipefd[1]);
       char buf[256];
       read(pipefd[0], buf, sizeof(buf));
       close(pipefd[0]);
   } else {
       // Parent process: ghi vào pipe
       close(pipefd[0]);
       write(pipefd[1], "Hello from parent", 18);
       close(pipefd[1]);
   }

Ưu điểm:

- Đơn giản
- Nhanh chóng
- An toàn

Hạn chế:

- Lưu lượng giới hạn
- Chỉ truyền được một chiều (half-duplex)
- Không có cơ chế đồng bộ

**shared memory**

Cho phép một vùng nhớ được cùng truy cập bởi nhiều process cùng lúc.

.. code-block:: c

   #include <sys/shm.h>

   key_t key = ftok("/tmp", 'A');
   int shmid = shmget(key, 1024, IPC_CREAT | 0666);
   void *data = shmat(shmid, NULL, 0);

   // Ghi dữ liệu vào shared memory
   strcpy((char *)data, "Shared data");

   shmdt(data);  // Detach

Ưu điểm:

- Tiết kiệm bộ nhớ (không cần lưu riêng rẽ, phân mảnh cùng 1 dữ liệu tại
  nhiều local space của process)
- Truy cập vùng nhớ dễ dàng giữa các data khi setup
- Chia sẻ dữ liệu nhanh hơn IPC, tránh được overhead (write in wait)

Hạn chế:

- Khó implement, vì các process cần đồng bộ và quyền truy cập khác nhau
- Memory leak khi last hold process không free đúng cách
- Over wait ⇒ Deadlock (cần tracking bằng watchdog timer)

**semaphore**

Cờ hiệu, để giải quyết vấn đề đồng bộ của shared memory. Semaphore là
một cờ dạng int giúp Linux đồng bộ giữa các process.

.. code-block:: c

   #include <sys/sem.h>

   key_t key = ftok("/tmp", 'B');
   int semid = semget(key, 1, IPC_CREAT | 0666);
   semctl(semid, 0, SETVAL, 1);  // Khởi tạo giá trị = 1

   struct sembuf sb;
   sb.sem_num = 0;
   sb.sem_op = -1;  // Wait (P) - giảm semaphore
   semop(semid, &sb, 1);

   // Truy cập critical section...

   sb.sem_op = 1;   // Signal (V) - tăng semaphore
   semop(semid, &sb, 1);

Semaphore cung cấp cơ chế kiểm soát đơn giản:

- Tối ưu hóa IPC
- Tránh được race condition, sử dụng quá mức resource (excessive resource
  utilization)

Giá trị của một semaphore thường có 2 chiều:

- Tăng = signal/post
- Giảm = wait/acquire

Với positive int, process được cho phép truy cập vào space để thao tác
với vùng nhớ. Ngược lại, negative int, process phải chờ do space đang
bị block/busy.

**message queue**

Phương thức truyền tải dữ liệu IPC phổ biến nhất, thường được triển khai
trên các service đặc biệt là application layer (platform).

Message queue là một chuỗi dữ liệu được lưu trữ dưới dạng linked-list
và lưu trong kernel. Cấu trúc dữ liệu này theo kiểu FIFO và các tin nhắn
được xác định dựa trên Message ID.

.. code-block:: c

   #include <sys/msg.h>

   key_t key = ftok("/tmp", 'C');
   int msqid = msgget(key, IPC_CREAT | 0666);

   struct msgbuf {
       long mtype;       // Message type (> 0)
       char mtext[256];  // Message data
   } msg;

   // Gửi message
   msg.mtype = 1;
   strcpy(msg.mtext, "Hello via message queue");
   msgsnd(msqid, &msg, sizeof(msg.mtext), 0);

   // Nhận message
   msgrcv(msqid, &msg, sizeof(msg.mtext), 1, 0);

   // Xóa message queue
   msgctl(msqid, IPC_RMID, NULL);

Các API chính:

- ``ftok()`` → tạo unique key
- ``msgget()`` → lấy hoặc tạo mới message queue
- ``msgsnd()`` → thêm message vào cuối chuỗi
- ``msgrcv()`` → truy xuất messages từ một chuỗi
- ``msgctl()`` → delete message

POSIX (Portable Operating System Interface)
-------------------------------------------

- pthread
- mutex
- condition variable

POSIX là các interface tách biệt hoàn toàn với OS, platform layer.

**pthread**

Interface cho phép process tạo ra nhiều thread khác nhau để concurrency
handle process flow, tận dụng được multithread trên các thiết bị hiện nay.
Thư viện ``<pthread.h>``.

.. code-block:: c

   #include <pthread.h>

   void *thread_func(void *arg) {
       printf("Thread running, arg = %s\n", (char *)arg);
       return NULL;
   }

   int main() {
       pthread_t tid;
       char *msg = "Hello from main";

       // Tạo thread
       pthread_create(&tid, NULL, thread_func, msg);

       // Chờ thread kết thúc
       pthread_join(tid, NULL);

       return 0;
   }

Các API quan trọng trong ``<pthread.h>``:

- ``pthread_create()`` - Tạo thread mới
- ``pthread_join()`` - Chờ thread kết thúc
- ``pthread_detach()`` - Tách thread (tự động giải phóng khi kết thúc)
- ``pthread_exit()`` - Kết thúc thread hiện tại
- ``pthread_self()`` - Lấy ID của thread hiện tại
- ``pthread_equal()`` - So sánh hai thread IDs
- ``pthread_cancel()`` - Gửi yêu cầu hủy thread
- ``pthread_setcanceltype()`` - Thiết lập kiểu hủy (async/deferred)
- ``pthread_once()`` - Đảm bảo một hàm chỉ chạy một lần
- ``pthread_key_create()`` - Tạo thread-specific data key
- ``pthread_setspecific()`` - Gán dữ liệu riêng cho thread
- ``pthread_getspecific()`` - Lấy dữ liệu riêng của thread

**mutex**

Một cơ chế khóa, đồng bộ giữa các process trên application layer, xuất
hiện thường xuyên trong các ứng dụng multi-thread hoặc realtime process.

.. code-block:: c

   #include <pthread.h>

   pthread_mutex_t mutex = PTHREAD_MUTEX_INITIALIZER;
   int shared_counter = 0;

   void *increment(void *arg) {
       for (int i = 0; i < 1000000; i++) {
           pthread_mutex_lock(&mutex);
           shared_counter++;  // Critical section
           pthread_mutex_unlock(&mutex);
       }
       return NULL;
   }

Việc lập trình đa nhiệm mà không sử dụng mutex có thể dẫn tới các vấn đề:

- **Deadlock**: Hai thread cùng chờ nhau giải phóng tài nguyên
- **Race condition**: Kết quả phụ thuộc vào thứ tự thực thi không xác định
- **Unpredicted result**: Dữ liệu sai lệch do truy cập đồng thời

Các trường hợp sử dụng mutex:

1. **Bảo vệ shared data**: Khi nhiều thread cùng đọc/ghi một biến toàn cục
2. **Atomic operations**: Đảm bảo một chuỗi thao tác được thực thi liên tục
3. **Producer-Consumer**: Đồng bộ giữa thread sản xuất và tiêu thụ dữ liệu
4. **Resource pool**: Quản lý truy cập vào pool tài nguyên dùng chung

Những điều cần tránh khi sử dụng mutex:

1. **Deadlock**: Không bao giờ khóa nhiều mutex theo thứ tự khác nhau ở
   các thread khác nhau. Luôn lock theo cùng một thứ tự.
2. **Không unlock**: Luôn unlock mutex sau khi hoàn thành critical section.
   Sử dụng ``pthread_mutex_destroy()`` khi không còn dùng nữa.
3. **Recursive lock**: Mặc định mutex không cho phép cùng một thread lock
   lại chính nó (gây deadlock). Dùng ``PTHREAD_MUTEX_RECURSIVE`` nếu cần.
4. **Spin-lock trên mutex**: Tránh giữ mutex quá lâu, gây lãng phí CPU.
5. **Lock ordering violation**: Luôn lock/unlock theo thứ tự cố định để
   tránh deadlock.

**condition variable**

Cơ chế đồng bộ cho phép một thread chờ đợi một điều kiện cụ thể xảy ra,
thay vì phải busy-wait (polling). Condition variable luôn đi kèm với mutex.

.. code-block:: c

   #include <pthread.h>

   pthread_mutex_t mutex = PTHREAD_MUTEX_INITIALIZER;
   pthread_cond_t cond = PTHREAD_COND_INITIALIZER;
   int data_ready = 0;

   void *producer(void *arg) {
       // Sản xuất dữ liệu...
       pthread_mutex_lock(&mutex);
       data_ready = 1;
       pthread_cond_signal(&cond);  // Báo hiệu cho consumer
       pthread_mutex_unlock(&mutex);
       return NULL;
   }

   void *consumer(void *arg) {
       pthread_mutex_lock(&mutex);
       while (!data_ready) {
           // Tự động unlock mutex và chờ signal
           pthread_cond_wait(&cond, &mutex);
           // Khi được đánh thức, tự động lock lại mutex
       }
       // Xử lý dữ liệu...
       pthread_mutex_unlock(&mutex);
       return NULL;
   }

Các API chính:

- ``pthread_cond_wait()`` - Chờ điều kiện (unlock mutex, chờ signal, lock lại)
- ``pthread_cond_signal()`` - Đánh thức một thread đang chờ
- ``pthread_cond_broadcast()`` - Đánh thức tất cả thread đang chờ
- ``pthread_cond_timedwait()`` - Chờ có timeout

Lưu ý: Luôn kiểm tra điều kiện trong vòng lặp ``while`` (không dùng ``if``)
để tránh *spurious wakeup* (thread bị đánh thức giả).

Project
-------

- ✅ Viết mini shell
- ✅ TCP server
- ✅ UART terminal

**Mini shell**

Viết một shell đơn giản sử dụng ``fork()`` + ``exec()`` để thực thi lệnh,
hỗ trợ pipe (``|``) và redirection (``>``, ``<``).

.. code-block:: c

   // Pseudocode
   while (1) {
       print_prompt();
       read_command(cmd);
       if (fork() == 0) {
           // Child: thực thi lệnh
           execvp(cmd.argv[0], cmd.argv);
       } else {
           // Parent: chờ child kết thúc
           wait(NULL);
       }
   }

**TCP server**

Viết TCP server sử dụng socket API, hỗ trợ multiple clients với
``epoll()`` hoặc ``pthread``.

.. code-block:: c

   // Pseudocode
   int server_fd = socket(AF_INET, SOCK_STREAM, 0);
   bind(server_fd, ...);
   listen(server_fd, 5);

   while (1) {
       int client_fd = accept(server_fd, ...);
       // Xử lý client trong thread riêng
       pthread_create(&tid, NULL, handle_client, &client_fd);
   }

**UART terminal**

Viết chương trình giao tiếp UART sử dụng ``open()``, ``read()``,
``write()``, ``ioctl()`` trên ``/dev/ttyUSB0`` hoặc ``/dev/ttyS0``.

.. code-block:: c

   // Pseudocode
   int uart_fd = open("/dev/ttyUSB0", O_RDWR | O_NOCTTY);
   struct termios tty;
   tcgetattr(uart_fd, &tty);
   cfsetospeed(&tty, B115200);
   cfsetispeed(&tty, B115200);
   tcsetattr(uart_fd, TCSANOW, &tty);

   write(uart_fd, "AT\r\n", 4);
   char resp[256];
   read(uart_fd, resp, sizeof(resp));