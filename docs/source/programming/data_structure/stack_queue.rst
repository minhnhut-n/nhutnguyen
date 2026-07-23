Stacks & Queues
===============

Stack (Ngăn xếp) — LIFO (Last In, First Out)
Queue (Hàng đợi) — FIFO (First In, First Out)

Priority Queue (Hàng đợi ưu tiên)
---------------------------------

**What is this? (Đây là gì?)**
Priority Queue là cấu trúc dữ liệu mà mỗi phần tử có một độ ưu tiên. Phần tử có độ ưu tiên cao nhất luôn được lấy ra trước, bất kể thứ tự chèn vào. Triển khai phổ biến nhất là dùng **heap nhị phân** (binary heap) — cấu trúc dạng cây, giống binary tree nhưng không phải binary tree.

Một priority queue phải đảm bảo:
- Phần tử có độ ưu tiên cao hơn được sắp xếp trên đầu → search min/max O(1)
- Thứ tự ưu tiên được xếp theo tầng trong cấu trúc dạng tree
- Bất kể thời điểm chèn vào, các node luôn được sắp xếp đúng thứ tự ưu tiên

**Where can it be used? (Có thể dùng ở đâu?)**
- Dijkstra / A* — lấy node có khoảng cách nhỏ nhất để duyệt tiếp
- Task scheduling — hệ điều hành xếp lịch tiến trình theo độ ưu tiên
- Event-driven simulation — xử lý sự kiện theo thời gian
- Data streaming — tìm top K phần tử lớn nhất/nhỏ nhất trong luồng dữ liệu
- Huffman coding — xây dựng cây nén

**Which circumstance to use? (Dùng trong hoàn cảnh nào?)**
- Khi cần xử lý phần tử quan trọng nhất trước, không phải theo thứ tự chèn
- Khi cần thường xuyên lấy ra phần tử min/max từ một tập hợp động
- Khi dữ liệu liên tục được thêm vào và cần truy vấn phần tử ưu tiên cao nhất

**How to use? (Cách dùng?)**
1. Khởi tạo heap rỗng
2. Push phần tử vào heap (kèm độ ưu tiên nếu cần)
3. Pop phần tử có độ ưu tiên cao nhất (min hoặc max)
4. Heap tự động sắp xếp lại sau mỗi lần push/pop

**When to use? (Khi nào dùng?)**
- Khi bài toán có thể tối ưu bằng cách luôn xử lý "tốt nhất" trước
- Khi cần độ phức tạp O(log n) cho insert và O(1) cho lấy min/max
- Khi sort toàn bộ dữ liệu là quá tốn kém (O(n log n)) nhưng chỉ cần top K

**Ví dụ:** Tìm ra 100 số lớn nhất trong tổng số 1 triệu số.
Thay vì sort 1 triệu số (O(n log n)), ta tạo một min-heap size 100.
Duyệt 1 triệu số, nếu số lớn hơn min trong heap thì push vào và pop min đi.
Mỗi lần push/pop tốn O(log 100) ≈ O(1). Tổng: O(n log k) với k = 100.

.. code-block:: python

   import heapq

   # Min-heap (mặc định) — ưu tiên số nhỏ nhất
   pq = []
   heapq.heappush(pq, 5)
   heapq.heappush(pq, 1)
   heapq.heappush(pq, 3)
   print(heapq.heappop(pq))  # 1

   # Max-heap (dùng negative value) — ưu tiên số lớn nhất
   max_pq = []
   heapq.heappush(max_pq, -5)
   heapq.heappush(max_pq, -1)
   heapq.heappush(max_pq, -3)
   print(-heapq.heappop(max_pq))  # 5

   # Top 100 largest numbers from 1 million
   def top_k_largest(nums, k):
       heap = []
       for num in nums:
           if len(heap) < k:
               heapq.heappush(heap, num)
           elif num > heap[0]:
               heapq.heapreplace(heap, num)
       return heap