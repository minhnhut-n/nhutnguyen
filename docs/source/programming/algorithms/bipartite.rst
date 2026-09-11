Bipartite Graph (Đồ thị hai phía)
=================================

Kiểm tra tính hai phía (2-coloring) và ghép cặp tối đa (maximum matching)
trên **đồ thị hai phía** — một trong những cấu trúc đồ thị hữu ích nhất
trong bài toán gán cặp (assignment).

**What is this? (Đây là gì?)**
Đồ thị hai phía là đồ thị mà tập đỉnh chia được thành hai tập rời nhau
``U`` và ``V`` sao cho **mọi cạnh đều nối một đỉnh thuộc ``U`` với một
đỉnh thuộc ``V``** — không tồn tại cạnh nào nối hai đỉnh trong cùng một tập.

.. code-block:: text

       U1 --- V1        Mọi cạnh: U -> V
       U2 --- V1        Không có cạnh U -> U hay V -> V
       U2 --- V2
       U3 --- V2

**Where can it be used? (Có thể dùng ở đâu?)**
- Gán công việc: N công việc cho M nhân viên, mỗi người chỉ làm được một số việc
- Pairing: sinh viên — phòng ký túc xá, mentor — mentee, khách — bàn ăn
- Matchmaking trong game, hệ thống ghép cặp người chơi
- Hệ thống khuyến nghị (user — item), spam detection
- Lập lịch thi: gán môn học — phòng thi sao cho không trùng

**Which circumstance to use? (Dùng trong hoàn cảnh nào?)**
- Bài toán có 2 nhóm đối tượng, chỉ có quan hệ "khớp" giữa 2 nhóm
- Cần ghép cặp tối đa (maximum matching) hoặc ghép sao cho không ai trùng nhau
- Cần kiểm tra một đồ thị có chia được thành 2 nhóm "không có cạnh nội bộ"

**How to use? (Cách dùng?)**
1. Chọn một đỉnh bất kỳ, tô màu ``0``
2. BFS/DFS lan truyền sang các neighbor, tô màu **ngược lại** (``1 - color``)
3. Nếu gặp đỉnh đã tô **trùng màu** với đỉnh hiện tại → đồ thị **KHÔNG bipartite**
4. Nếu tô xong toàn bộ thành công → đồ thị bipartite

**When to use? (Khi nào dùng?)**
- Đồ thị **không có trọng số** trên cạnh (dạng thuần cấu trúc)
- Số đỉnh lớn, cần thuật toán hiệu quả: kiểm tra bipartite là ``O(V + E)``
- Cần ghép cặp tối đa: dùng **Kuhn/Hungarian** ``O(V × E)`` hoặc **Hopcroft–Karp** ``O(E × √V)``

---

Kiểm tra tính Bipartite (2-Coloring bằng BFS)
---------------------------------------------

Định lý (**Kőnig, 1936**): *Một đồ thị là bipartite ⇔ đồ thị không chứa
chu trình độ dài lẻ (odd cycle).*

.. code-block:: python

   from collections import deque

   def is_bipartite(graph, n):
       """
       graph: danh sách kề — graph[u] = [v, ...]
       n: số đỉnh (đánh số 0 .. n-1)
       Trả về: (True, color) nếu bipartite, ngược lại (False, None)
       """
       color = [-1] * n
       for start in range(n):
           if color[start] != -1:
               continue
           color[start] = 0
           queue = deque([start])
           while queue:
               u = queue.popleft()
               for v in graph[u]:
                   if color[v] == -1:
                       color[v] = 1 - color[u]
                       queue.append(v)
                   elif color[v] == color[u]:
                       return False, None   # hai đỉnh kề cùng màu
       return True, color

.. note::
   Với **đồ thị có trọng số**, một đồ thị là bipartite ⇔ mọi chu trình
   có **tổng trọng số lẻ** (trường hợp đặc biệt: mỗi cạnh thay đổi dấu
   qua từng bước — kỹ thuật *Parity Coloring*).

---

Maximum Matching — Ghép cặp tối đa (Kuhn's Algorithm)
-----------------------------------------------------

**What is this? (Đây là gì?)**
Tìm số cặp ghép được **nhiều nhất** giữa ``U`` và ``V`` sao cho mỗi đỉnh
chỉ thuộc **một** cặp. Thuật toán Kuhn (Hungarian) giải quyết bài toán
này bằng kỹ thuật **augmenting path**.

**Định nghĩa quan trọng:**

- **Matching:** tập các cạnh ghép cặp, không hai cạnh nào chia sẻ chung một đỉnh
- **Augmenting path:** đường đi bắt đầu từ đỉnh *chưa ghép* của ``U``,
  xen kẽ cạnh *chưa ghép / đã ghép*, và kết thúc tại đỉnh *chưa ghép* của ``V``.
  Tìm được augmenting path → số cặp ghép tăng thêm 1.

.. code-block:: python

   def kuhn_matching(graph, n_left, n_right):
       """
       graph: danh sách kề — graph[u] = [v, ...] với u thuộc tập U
       Trả về: match — match[v] = u nghĩa là ghép u với v
       """
       match = [-1] * n_right   # đỉnh V đang ghép với đỉnh U nào

       def try_kuhn(u, visited):
           for v in graph[u]:
               if not visited[v]:
                   visited[v] = True
                   if match[v] == -1 or try_kuhn(match[v], visited):
                       match[v] = u
                       return True
           return False

       for u in range(n_left):
           visited = [False] * n_right
           try_kuhn(u, visited)
       return match

**Độ phức tạp:** ``O(V × E)`` — chấp nhận được khi số cạnh vừa phải
(nhỏ hơn ~10⁵).

---

Hopcroft–Karp — Ghép cặp tối đa nhanh hơn
-----------------------------------------

**What is this? (Đây là gì?)**
Phiên bản tối ưu của Kuhn: thay vì tìm augmenting path **từng cái một**,
Hopcroft–Karp tìm **một tập các augmenting path ngắn nhất** mỗi pha.
Số pha chỉ là ``O(√V)`` → tổng độ phức tạp ``O(E × √V)``.

**When to use? (Khi nào dùng?)**
- Đồ thị lớn: số cạnh tới ~10⁵–10⁶ (thuật toán competitive programming)
- Kuhn quá chậm, cần tối ưu về thời gian

.. code-block:: python

   from collections import deque

   INF = float('inf')

   def hopcroft_karp(graph, n_left, n_right):
       """
       graph: danh sách kề — graph[u] = [v, ...] với u thuộc tập U
       Trả về: số cặp ghép tối đa
       """
       match_u = [-1] * n_left    # mỗi đỉnh U ghép với đỉnh V nào
       match_v = [-1] * n_right   # mỗi đỉnh V ghép với đỉnh U nào
       dist = [0] * n_left

       def bfs():
           queue = deque()
           for u in range(n_left):
               if match_u[u] == -1:
                   dist[u] = 0
                   queue.append(u)
               else:
                   dist[u] = INF
           found = False
           while queue:
               u = queue.popleft()
               for v in graph[u]:
                   nxt = match_v[v]
                   if nxt == -1:
                       found = True
                   elif dist[nxt] == INF:
                       dist[nxt] = dist[u] + 1
                       queue.append(nxt)
           return found

       def dfs(u):
           for v in graph[u]:
               nxt = match_v[v]
               if nxt == -1 or (dist[nxt] == dist[u] + 1 and dfs(nxt)):
                   match_u[u] = v
                   match_v[v] = u
                   return True
           dist[u] = INF
           return False

       matching = 0
       while bfs():
           for u in range(n_left):
               if match_u[u] == -1 and dfs(u):
                   matching += 1
       return matching


---

Định lý Kőnig & Bài toán Vertex Cover
-------------------------------------

**Định lý Kőnig (Kőnig–Egerváry):** Trong đồ thị hai phía,
số cặp ghép tối đa = số đỉnh ít nhất cần chọn để "che" toàn bộ cạnh
(**minimum vertex cover**).

Ứng dụng phổ biến: bài toán *"chọn ít ô/đỉnh nhất để đánh dấu toàn bộ
các cạnh"* — ví dụ gạch chân các hàng/cột trong ma trận để che hết
các phần tử bị cấm.

.. code-block:: text

   Maximum Matching = Minimum Vertex Cover (chỉ đúng trên đồ thị bipartite)

---

Tổng kết độ phức tạp
====================

.. list-table:: **Độ phức tạp các thuật toán trên đồ thị hai phía**
   :widths: 35 25 40
   :header-rows: 1

   * - Thuật toán
     - Độ phức tạp
     - Mục đích
   * - BFS/DFS 2-Coloring
     - ``O(V + E)``
     - Kiểm tra đồ thị có bipartite
   * - Kuhn (Hungarian)
     - ``O(V × E)``
     - Maximum matching, đồ thị vừa phải
   * - Hopcroft–Karp
     - ``O(E × √V)``
     - Maximum matching, đồ thị lớn
   * - Hungarian (weighted, Kuhn–Munkres)
     - ``O(V³)``
     - Ghép cặp tối đa **trọng số**

.. tip::
   Bài toán thực tế thường giải theo 2 bước:

   #. Mô hình hóa bài toán thành đồ thị hai phía (xác định tập ``U``, ``V`` và cạnh)
   #. Kiểm tra bipartite (nếu cần), rồi áp dụng maximum matching

