[UDEMY] Object Pattern trong Lập Trình C
========================================

.. role:: bolditalic
   :class: bolditalic

**Object Pattern** là nền tảng quan trọng trong lập trình C, giúp tổ chức code theo hướng hướng đối tượng mà không cần sử dụng OOP truyền thống.

**Object Pattern** trong ngôn ngữ C là việc gom nhóm một tập hợp các dữ liệu thành một tổ chức có cấu trúc (hierarchical structure) đi kèm với các hàm (functions) thao tác/sửa đổi (modify) chính các cấu trúc dữ liệu đó.

**Tóm tắt:** Thay vì quản lý dữ liệu rời rạc, ta gom chúng vào ``struct`` và cung cấp các hàm để thao tác an toàn.

Nguyên tắc chính:

  * **Xử lý theo luồng (Data Flow):** Dữ liệu được xử lý rõ ràng theo luồng, tránh tham chiếu (reference) đến các giá trị nằm ngoài phạm vi (scope) của hàm.
  * **Tính nhất quán (Thread Safety/Reentrancy):** Dựa trên nguyên tắc này, nếu 2 thread cùng gọi một hàm với cùng một đầu vào (*input*), dữ liệu đầu ra (*output*) thu được sẽ hoàn toàn như nhau.
  * **Sạch mã nguồn (Clean Code):** Mang lại lợi ích vô cùng to lớn (*tremendous*) cho việc duy trì và quản lý code. Khi dữ liệu được thay đổi, nó sẽ thực hiện trực tiếp thông qua một hàm xác định, vì dữ liệu nằm chính trong *Object* và được truyền dưới dạng đối số (*argument*) vào trong từng hàm.

Quy tắc cốt lõi:

  1. **Context** được truyền xuyên suốt thông qua các đối số (*arguments / parameters*).
  2. **Dữ liệu** tuyệt đối không được phép truy cập toàn cục (*globally*).
  3. **Function** không nên chứa biến tĩnh (*static variables*).
  4. **Data flow** đi theo một luồng duy nhất và thống nhất.

**Ví dụ minh họa:**

.. code-block:: c

   // ❌ Bad: Global variables
   int g_counter;
   
   void increment(void) {
       g_counter++;  // Không rõ ràng, khó debug
   }

   // ✅ Good: Object Pattern
   typedef struct {
       int counter;
   } Counter;
   
   void counter_increment(Counter *self) {
       self->counter++;  // Rõ ràng, dễ test
   }
