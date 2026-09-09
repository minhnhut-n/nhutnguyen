Searching algorithms
====================

Tài liệu về các thuật toán tìm kiếm: tìm kiếm tuyến tính, tìm kiếm nhị phân, và các biến thể.


===============================================================================
Thuật toán tìm kiếm theo chiều rộng (breadth-first search - BFS)
===============================================================================

.. meta::
   :description: Tài liệu chi tiết về thuật toán Breadth-First Search (BFS), ý tưởng cốt lõi, thứ tự thực hiện và sơ đồ mô phỏng ma trận kề.
   :keywords: BFS, Breadth First Search, Graph Theory, Algorithm, Queue, Adjacency Matrix

---

1. Tổng quan về Thuật toán
--------------------------

**Breadth-First Search (BFS)** là thuật toán tìm kiếm/duyệt cây hoặc đồ thị theo chiều rộng.

Khái niệm Kỹ thuật
~~~~~~~~~~~~~~~~~~
* Thuật toán bắt đầu từ một **node nguồn (Start Node)** cố định, sau đó lan rộng ra theo **chiều ngang** để khám phá từng cấp (level/layer) của đồ thị.
* BFS luôn ưu tiên **thăm tất cả các node kề trực tiếp** với node hiện tại trước, rồi mới tiếp tục chuyển sang các node lân cận ở cấp tiếp theo.

.. note::
   Tính chất duyệt theo từng lớp của BFS giúp nó trở thành giải pháp tối ưu cho bài toán **tìm đường đi ngắn nhất** (Shortest Path) trên đồ thị không có trọng số.

---

2. Ý tưởng Cốt lõi & Cấu trúc Dữ liệu
-------------------------------------

Thuật toán duy trì trạng thái bằng 2 cấu trúc dữ liệu chính:

* **Hàng chờ (Queue):** 
  Làm nơi lưu trữ tạm thời các đỉnh chờ được duyệt. Cấu trúc này tuân theo nguyên lý **FIFO (First In, First Out)** — đỉnh nào được đưa vào trước sẽ được xử lý trước.
* **Mảng / Tập hợp đánh dấu (Visited Array / Set):** 
  Lưu trữ trạng thái (đã duyệt hay chưa) tương ứng với danh sách đỉnh của đồ thị, giúp tránh rơi vào vòng lặp vô tận (Cycle) khi duyệt đồ thị.

---

3. Thứ tự Thực hiện Thuật toán (Execution Steps)
------------------------------------------------

.. code-block:: text

   +-------------------+      +--------------------+      +--------------------+
   | 1. Chọn Start Node| ---> | 2. Enqueue & Mark  | ---> | 3. Pop Front Node  |
   +-------------------+      +--------------------+      +--------------------+
                                                                    |
   +-------------------+      +--------------------+                v
   | 6. KẾT THÚC      | <--- | 5. KTra Queue Rỗng | <--- | 4. Duyệt Node Kề   |
   +-------------------+      +--------------------+      +--------------------+

Các bước triển khai từng bước:

1. **Khởi tạo:** Chọn một đỉnh (Vertex) làm điểm xuất phát.
2. **Kích hoạt:** Đưa đỉnh xuất phát vào Queue và đánh dấu trạng thái là **Đã duyệt (Visited)**.
3. **Lấy dữ liệu (Pop / Dequeue):** Lấy đỉnh nằm ở đầu Queue (Top/Front of Queue) ra để truy cập dữ liệu.
4. **Duyệt lân cận:** Thăm tất cả các node kề với node vừa lấy ra mà **chưa được đánh dấu**:
   * Đánh dấu các node kề này là **Đã duyệt**.
   * Đưa (Enqueue) các node kề này vào Queue.
5. **Lặp lại:** Tiếp tục thực hiện rút đỉnh (Dequeue) và lặp lại bước 4.
6. **Kết thúc:** Thuật toán dừng lại ngay khi Queue trở thành **Rỗng (Empty)**.

---

4. Mô phỏng Đồ thị & Ma trận Kề (Adjacency Matrix)
--------------------------------------------------

Sơ đồ Đồ thị 5 Đỉnh (Graph Structure)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Đồ thị vô hướng gồm 5 đỉnh (vertices) từ ``0`` đến ``4``:

.. code-block:: text

        (0)-------------* (1)
         |                 |
         |                 |
        *|                 |
        (3)------------- (2)
          \               /
           \             /
            \           /
             --- (4) ---

* **Danh sách các đỉnh (Vertices):** ``{0, 1, 2, 3, 4}``

Ma trận Kề (Adjacency Matrix 5x5)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Biểu diễn mối quan hệ kết nối giữa các node (``1``: Có cạnh nối, ``0``: Không có cạnh nối):

.. list-table:: **Ma trận kề 5x5**
   :widths: 20 16 16 16 16 16
   :header-rows: 1
   :stub-columns: 1
   :align: center

   * - Node
     - 0
     - 1
     - 2
     - 3
     - 4
   * - **0**
     - 1
     - 1
     - 0
     - 1
     - 0
   * - **1**
     - 1
     - 1
     - 1
     - 0
     - 0
   * - **2**
     - 0
     - 1
     - 1
     - 1
     - 1
   * - **3**
     - 1
     - 0
     - 1
     - 1
     - 1
   * - **4**
     - 0
     - 0
     - 1
     - 1
     - 1

---

5. Biểu diễn Chi tiết Các bước Duyệt qua Queue (Fronts)
-------------------------------------------------------

Mã giả Thuật toán (Pseudo Code)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: python

   def bfs(start_idx, nums, adjacency_matrix):
       mark_checked[start_idx] = 1
       # Xếp hàng start_idx vào queue
       
       while (còn node trong mark_checked chưa được pop):
           # Truy cập node đó
           # Kiểm tra node kề trong matrix và chưa được checked:
               cập nhật adjacency_matrix
               mark_checked[j] = node_new
               
       # Điểm dừng thuật toán
       if rear < front:
           exit

Sơ đồ Dịch chuyển Con trỏ Fronts (Fronts Matrix Diagram)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Sơ đồ mô phỏng vị trí con trỏ ``front`` dịch chuyển qua từng bước duyệt node trong Queue:

.. code-block:: text

       front 1   front 2   front 3   front 4   front 5
          |         |         |         |         |
          v         v         v         v         v
        +---+     +---+     +---+     +---+     +---+
    0   | 1 |---->| 1 |     | 0 |     | 1 |     | 0 |
        +---+     +---+     +---+     +---+     +---+
    1   | 1 |     | 1 |---->| 1 |     | 0 |     | 0 |
        +---+     +---+     +---+     +---+     +---+
    2   | 0 |     | 1 |     | 1 |---->| 1 |     | 1 |
        +---+     +---+     +---+     +---+     +---+
    3   | 1 |     | 0 |     | 1 |     | 1 |---->| 1 |
        +---+     +---+     +---+     +---+     +---+
    4   | 0 |     | 0 |     | 1 |     | 1 |     | 1 |  <-- exit khi rear < front
        +---+     +---+     +---+     +---+     +---+

Sơ đồ Hướng duyệt theo Đường chéo (Traversal Directions)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Về cơ bản, với từng mốc theo đường chéo, thuật toán BFS sẽ duyệt qua các cặp được phối hợp từ hai hàng ngang - dọc, cùng với mốc chéo tiếp theo. Quá trình tiếp diễn cho đến khi đạt kết quả cuối cùng là không còn giá trị nào để pop trong queue:

.. code-block:: text

   +---------------------------------------+
   |  \                                    |
   |   +---------------------------------> |
   |   |  \                                |
   |   |   +-----------------------------> |
   |   |   |  \                            |
   |   |   |   +-------------------------> |
   |   v   v   v                           |
   +---------------------------------------+

===============================================================================
thuật toán tìm kiem theo chiều sâu (depth-first search - DFS)
===============================================================================

1. Tổng quan về Thuật toán DFS
------------------------------

**Depth-First Search (DFS)** là thuật toán tìm kiếm/duyệt cây hoặc đồ thị theo **chiều sâu**.

Khái niệm Kỹ thuật
~~~~~~~~~~~~~~~~~~
* Thuật toán bắt đầu từ một **node nguồn (Start Node)** và tiến hành đi sâu nhất có thể theo một nhánh duy nhất (*Dive to deep*).
* Khi chạm tới đỉnh tận cùng (không còn node lân cận nào chưa duyệt), thuật toán sẽ **quay lùi (Backtrack)** về nút giao gần nhất để thử sang một nhánh mới.

.. note::
   Khác với BFS duyệt theo cơ chế "vết dầu loang", DFS di chuyển như việc đi qua một mê cung: đi sâu vào một con đường cho tới khi gặp đường cụt rồi mới quay lại chọn đường khác.

---

2. So sánh BFS vs DFS
----------------------

Bảng dưới đây tổng hợp các điểm khác biệt cốt lõi giữa hai thuật toán duyệt đồ thị kinh điển:

.. list-table:: **Bảng so sánh chi tiết BFS và DFS**
   :widths: 20 40 40
   :header-rows: 1
   :align: center

   * - Tiêu chí
     - BFS (Breadth-First Search)
     - DFS (Depth-First Search)
   * - **Cấu trúc dữ liệu**
     - Hàng chờ **Queue** (FIFO - First In, First Out)
     - Ngăn xếp **Stack** (LIFO) hoặc **Đệ quy**
   * - **Chiến lược duyệt**
     - Duyệt từng cấp lân cận (*Vết dầu loang*)
     - Đi sâu hết một nhánh (*Dive to deep*)
   * - **Đặc điểm chính**
     - Khám phá các đỉnh theo chiều ngang
     - Khám phá kiệt cùng một nhánh rồi quay lùi (Backtrack)
   * - **Ưu thế bài toán**
     - Tìm đường đi ngắn nhất (Shortest Path)
     - Giải mê cung, phát hiện chu trình, sắp xếp Topo

---

3. Mã giả Thuật toán DFS (Pseudo Code)
--------------------------------------

Mã giả DFS triển khai bằng Đệ quy (Recursion)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: python

   def dfs_recursive(node, adjacency_matrix, visited):
       # Đánh dấu đỉnh hiện tại là đã duyệt
       visited[node] = True
       print(f"Thăm node: {node}")
       
       # Duyệt qua tất cả các đỉnh kề trong ma trận kề
       for neighbor in range(len(adjacency_matrix[node])):
           # Nếu có cạnh nối và đỉnh kề chưa được thăm
           if adjacency_matrix[node][neighbor] == 1 and not visited[neighbor]:
               # Đệ quy đi sâu vào đỉnh kề đó
               dfs_recursive(neighbor, adjacency_matrix, visited)

Mã giả DFS triển khai bằng Stack (Iterative)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: python

   def dfs_stack(start_node, adjacency_matrix):
       visited = [False] * len(adjacency_matrix)
       stack = [start_node]
       
       while len(stack) > 0:
           # Lấy đỉnh ở top of stack ra (LIFO)
           node = stack.pop()
           
           if not visited[node]:
               visited[node] = True
               print(f"Thăm node: {node}")
               
               # Đưa các node kề chưa thăm vào stack
               for neighbor in range(len(adjacency_matrix[node]) - 1, -1, -1):
                   if adjacency_matrix[node][neighbor] == 1 and not visited[neighbor]:
                       stack.append(neighbor)

---

4. Mô phỏng Sơ đồ Tiến trình Duyệt DFS
--------------------------------------

Sơ đồ Đồ thị Mẫu (5 Đỉnh)
~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: text

        (0)-------------* (1)
         |                 |
         |                 |
        *|                 |
        (3)------------- (2)
          \               /
           \             /
            \           /
             --- (4) ---

Sơ đồ Luồng di chuyển "Dive to Deep & Backtrack"
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Dưới đây là mô phỏng luồng đi sâu hết một nhánh và quay đầu (giả sử xuất phát từ nút ``0``):

.. code-block:: text

   [Bước 1] Dive in:   (0) ----> (1) ----> (2) ----> (4) ----> (3)
                                                                 |
   [Bước 2] Trạng thái: (3) Không còn node kề chưa thăm (Đường cụt!)
                                                                 |
   [Bước 3] Backtrack: (3) --quay lùi--> (4) --quay lùi--> (2) --quay lùi--> (0)
                                                                 |
   [Bước 4] Kết thúc: Tất cả đỉnh đã được đánh dấu trong visited[].

Mô phỏng Trạng thái Stack qua từng bước
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: text

     +---+         +---+         +---+         +---+         +---+
     | 3 |         | 4 |         | 2 |         | 1 |         | 0 |
     +---+         +---+         +---+         +---+         +---+
     Stack (1)     Stack (2)     Stack (3)     Stack (4)     Stack (5)
     [Start 0]     [Push 1]      [Push 2]      [Push 4]      [Push 3]
         |             |             |             |             |
         v             v             v             v             v
     Thăm (0)      Thăm (1)      Thăm (2)      Thăm (4)      Thăm (3) -> POP ALL & EXIT