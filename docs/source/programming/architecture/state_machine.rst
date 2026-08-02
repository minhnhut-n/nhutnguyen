III. State Machine: Mô Hình Trạng Thái
======================================

.. rubric:: 1. Giới Thiệu Về State Machine

State machine là một mô hình toán học, trong đó trạng thái của nó là xác định tại một thời điểm bất kì.

"State" có nghĩa là trạng thái, nó chuyển từ một trạng thái sang trạng thái khác để đáp ứng các yêu cầu từ inputs hoặc events.

Ví dụ điển hình của state machine là "đèn giao thông":

::

  ┌─────────────────────────────────────────┐
  │         TRAFFIC LIGHT STATE MACHINE     │
  └─────────────────────────────────────────┘
  
  ┌──────┐    Timer    ┌──────┐    Timer    ┌──────┐
  │ RED  │ ──────────► │GREEN │ ──────────► │YELLOW│
  │ (5s) │             │ (5s) │             │ (2s) │
  └──────┘             └──────┘             └──────┘
       ▲                                      │
       │                                      │
       └──────────────────────────────────────┘
              (Returns to RED, repeat cycle)

Thứ tự chuyển đổi lần lượt từ đỏ - xanh - vàng, mà không có ngoại lệ. Trạng thái của đèn tại một thời điểm cũng là xác định và không có trường hợp nào cả xanh và đỏ đồng thời.

*Hình vẽ minh họa: Đèn giao thông với các trạng thái tuần tự*

.. rubric:: 2. Ứng Dụng Trong Lập Trình

Trong lĩnh vực lập trình, state machine giúp ta giải quyết các vấn đề phức tạp một cách có cấu trúc, và dễ sử dụng, debug lỗi.

Quá trình chuyển đổi của các task trong state machine là tự động mà không cần phải sử dụng các cờ hiệu (bool, conditional logic) để quản lý/chuyển đổi giữa các task.

**So sánh thiết kế:**

- **Thiết Kế Tệ:** Sử dụng nhiều biến boolean và conditional logic phức tạp, dễ dẫn đến "flag explosion" và khó debug.
- **Thiết Kế Tốt:** Sử dụng state machine với các trạng thái rõ ràng, chuyển đổi tự động, dễ theo dõi và bảo trì.

**Ví dụ so sánh - Traffic Light Controller:**

::

  ❌ BAD DESIGN (Without State Machine):
  ┌──────────────────────────────────────────────┐
  │ bool is_red = true;                          │
  │ bool is_green = false;                       │
  │ bool is_yellow = false;                      │
  │ int timer = 0;                               │
  │                                              │
  │ void update() {                              │
  │     if (is_red && timer > 5) {               │
  │         is_red = false;                      │
  │         is_green = true;  // Bug: What if    │
  │         timer = 0;        // both are true?  │
  │     }                                        │
  │     // ... complex nested if-else            │
  │ }                                            │
  └──────────────────────────────────────────────┘

::

  ✅ GOOD DESIGN (With State Machine):
  ┌──────────────────────────────────────────────┐
  │ typedef enum {                               │
  │     STATE_RED,                               │
  │     STATE_GREEN,                             │
  │     STATE_YELLOW                             │
  │ } TrafficLightState;                         │
  │                                              │
  │ TrafficLightState current_state = STATE_RED; │
  │                                              │
  │ void transition() {                          │
  │     switch(current_state) {                  │
  │         case STATE_RED:                      │
  │             current_state = STATE_GREEN;     │
  │             break;                           │
  │         // Clear, explicit transitions       │
  │     }                                        │
  │ }                                            │
  └──────────────────────────────────────────────┘

.. rubric:: 3. Cấu Trúc Cốt Lõi (Anatomy)

State machine được cấu thành từ 5 thành tố cốt lõi để tạo ra một predictable behavior (hành vi có thể xác định được):

**1. State (Trạng Thái)**
- Đại diện cho một đối tượng (object), ta tạm gọi là "cột mốc"
- Trạng thái có thể là: running, idle, authenticated, v.v.

**2. Event (Sự Kiện)**
- Sự kiện xuất hiện tại mốc: ERROR, SUCCESS, REQUESTING, v.v.

**3. Transition (Chuyển Giao)**
- Định nghĩa trạng thái chuyển giao giữa các task khi một sự kiện liên quan xảy ra

**4. Initial State (Trạng Thái Ban Đầu)**
- Trạng thái khởi điểm của machine (ví dụ: đèn đỏ trước trong đèn giao thông)

**5. Final State (Trạng Thái Kết Thúc) - Optional**
- Trạng thái kết thúc của quy trình (nếu có)

**Sơ đồ cấu trúc tổng quan:**

::

  ┌─────────────────────────────────────────────────────┐
  │           STATE MACHINE STRUCTURE                    │
  └─────────────────────────────────────────────────────┘
  
  [Initial State]
       │
       │ (Start)
       ▼
  ┌─────────────┐
  │   STATE A   │◄──────────────┐
  │  (Idle)     │               │
  └─────────────┘               │
       │                         │
       │ Event: START            │ Event: RESET
       ▼                         │
  ┌─────────────┐               │
  │   STATE B   │               │
  │ (Running)   │               │
  └─────────────┘               │
       │                         │
       │ Event: COMPLETE         │
       ▼                         │
  ┌─────────────┐               │
  │   STATE C   │               │
  │ (Finished)  │               │
  └─────────────┘               │
       │                         │
       │ Event: DONE             │
       ▼                         │
  [Final State] ─────────────────┘

.. rubric:: 4. Triển Khai State Machine Design

**Quy trình triển khai:**

1. **Bắt đầu đơn giản:** Triển khai bằng một ngôn ngữ yêu thích (ưu tiên OOP language)

2. **Loại bỏ flags:** Cố gắng loại bỏ các flag, thủ tục chồng chéo nhau hết mức có thể để quay về đúng khái niệm state machine

3. **Nâng cao:** Sau khi hiểu về thiết kế state machine cơ bản, đi đến các khái niệm nâng cao:

   - **Hierarchical State (Nested States):** Các luồng làm việc lồng vào nhau, như mối quan hệ thư mục. Ví dụ: trong playing music có trạng thái điều chỉnh âm lượng, seeking (tìm bài), v.v.
   
   - **Parallel States:** Các luồng làm việc song song, cho phép nhiều state machine cùng chạy đồng thời với nhau

**Ví dụ minh họa - Media Player State Machine:**

::

  ┌──────────────────────────────────────────────────────┐
  │         MEDIA PLAYER - HIERARCHICAL STATES           │
  └──────────────────────────────────────────────────────┘
  
  ┌─────────────────┐
  │   PLAYING       │
  │   (Main State)  │
  └─────────────────┘
          │
          ├──► [Volume Control] ──► Volume Up / Down
          │         │
          │         └──► Mute / Unmute
          │
          ├──► [Seeking] ──► Forward / Backward
          │         │
          │         └──► Jump to Time
          │
          └──► [Playback Speed] ──► 1x / 1.5x / 2x

**Ví dụ code - Traffic Light FSM trong C:**

.. code-block:: c

  // State Machine Implementation for Traffic Light
  typedef enum {
      STATE_RED,
      STATE_GREEN,
      STATE_YELLOW
  } TrafficLightState;

  typedef struct {
      TrafficLightState current_state;
      int timer_seconds;
  } TrafficLightFSM;

  // Initialize FSM
  void fsm_init(TrafficLightFSM *fsm) {
      fsm->current_state = STATE_RED;
      fsm->timer_seconds = 0;
  }

  // Transition function
  void fsm_transition(TrafficLightFSM *fsm, Event event) {
      switch (fsm->current_state) {
          case STATE_RED:
              if (event == TIMER_EXPIRED && fsm->timer_seconds >= 5) {
                  fsm->current_state = STATE_GREEN;
                  fsm->timer_seconds = 0;
                  printf("Transition: RED → GREEN\n");
              }
              break;
              
          case STATE_GREEN:
              if (event == TIMER_EXPIRED && fsm->timer_seconds >= 5) {
                  fsm->current_state = STATE_YELLOW;
                  fsm->timer_seconds = 0;
                  printf("Transition: GREEN → YELLOW\n");
              }
              break;
              
          case STATE_YELLOW:
              if (event == TIMER_EXPIRED && fsm->timer_seconds >= 2) {
                  fsm->current_state = STATE_RED;
                  fsm->timer_seconds = 0;
                  printf("Transition: YELLOW → RED\n");
              }
              break;
      }
  }

  // Update function - called every second
  void fsm_update(TrafficLightFSM *fsm) {
      fsm->timer_seconds++;
  }

**Ví dụ code - Authentication State Machine trong Python:**

.. code-block:: python

  from enum import Enum
  from dataclasses import dataclass
  from typing import Optional

  class AuthState(Enum):
      UNAUTHENTICATED = "unauthenticated"
      AUTHENTICATING = "authenticating"
      AUTHENTICATED = "authenticated"
      SESSION_EXPIRED = "session_expired"

  class AuthEvent(Enum):
      LOGIN_REQUEST = "login_request"
      LOGIN_SUCCESS = "login_success"
      LOGIN_FAILURE = "login_failure"
      LOGOUT = "logout"
      SESSION_TIMEOUT = "session_timeout"

  @dataclass
  class AuthContext:
      username: Optional[str] = None
      token: Optional[str] = None
      retry_count: int = 0

  class AuthenticationFSM:
      def __init__(self):
          self.state = AuthState.UNAUTHENTICATED
          self.context = AuthContext()
          
      def transition(self, event: AuthEvent, data: dict = None):
          print(f"[{self.state.value}] Event: {event.value}")
          
          if self.state == AuthState.UNAUTHENTICATED:
              if event == AuthEvent.LOGIN_REQUEST:
                  self.state = AuthState.AUTHENTICATING
                  self.context.username = data.get('username')
                  print(f"  → Transitioning to AUTHENTICATING")
                  
          elif self.state == AuthState.AUTHENTICATING:
              if event == AuthEvent.LOGIN_SUCCESS:
                  self.state = AuthState.AUTHENTICATED
                  self.context.token = data.get('token')
                  print(f"  → Transitioning to AUTHENTICATED")
              elif event == AuthEvent.LOGIN_FAILURE:
                  self.context.retry_count += 1
                  if self.context.retry_count >= 3:
                      print(f"  → Max retries reached, staying in UNAUTHENTICATED")
                      self.state = AuthState.UNAUTHENTICATED
                  # else: stay in AUTHENTICATING state
                  
          elif self.state == AuthState.AUTHENTICATED:
              if event == AuthEvent.LOGOUT:
                  self.state = AuthState.UNAUTHENTICATED
                  self.context.token = None
                  print(f"  → Transitioning to UNAUTHENTICATED")
              elif event == AuthEvent.SESSION_TIMEOUT:
                  self.state = AuthState.SESSION_EXPIRED
                  print(f"  → Transitioning to SESSION_EXPIRED")
                  
          elif self.state == AuthState.SESSION_EXPIRED:
              if event == AuthEvent.LOGIN_REQUEST:
                  self.state = AuthState.AUTHENTICATING
                  print(f"  → Transitioning to AUTHENTICATING")

  # Demo usage
  if __name__ == "__main__":
      auth_fsm = AuthenticationFSM()
      
      # Scenario: Successful login
      auth_fsm.transition(AuthEvent.LOGIN_REQUEST, {"username": "user1"})
      auth_fsm.transition(AuthEvent.LOGIN_SUCCESS, {"token": "abc123"})
      
      # Scenario: Session timeout
      auth_fsm.transition(AuthEvent.SESSION_TIMEOUT)
      
      # Scenario: Re-login after expiry
      auth_fsm.transition(AuthEvent.LOGIN_REQUEST, {"username": "user1"})

**Output:**

::

  [unauthenticated] Event: login_request
    → Transitioning to AUTHENTICATING
  [authenticating] Event: login_success
    → Transitioning to AUTHENTICATED
  [authenticated] Event: session_timeout
    → Transitioning to SESSION_EXPIRED
  [session_expired] Event: login_request
    → Transitioning to AUTHENTICATING

.. rubric:: 5. Actions và Side Effects

Thiết kế state machine không chỉ là quản lý trạng thái, mà còn phải quản lý các side effects. Có 3 dạng actions chính:

**1. Entry Action**
- Được chạy khi join vào một trạng thái bất kì

**2. Exit Action**
- Chạy khi rời khỏi state, thường được sử dụng để cancel các requests và clear các thông tin tạm thời được sử dụng

**3. Transition Action**
- Chạy trong từng tình huống chuyển giao cụ thể, thường useful cho logging, phân tích, hoặc các conditional side effects

**Sơ đồ Actions trong State Lifecycle:**

::

  ┌─────────────────────────────────────────────────────┐
  │         STATE LIFECYCLE WITH ACTIONS                 │
  └─────────────────────────────────────────────────────┘
  
  [Previous State]
       │
       │ (Trigger Event)
       ▼
  ┌──────────────────┐
  │  EXIT ACTION     │ ← Cleanup, cancel requests
  │  (on_exit)       │
  └──────────────────┘
       │
       │ (Transition)
       ▼
  ┌──────────────────┐
  │ TRANSITION       │ ← Logging, validation
  │ ACTION           │
  └──────────────────┘
       │
       │ (Enter new state)
       ▼
  ┌──────────────────┐
  │  ENTRY ACTION    │ ← Initialize, start timers
  │  (on_entry)      │
  └──────────────────┘
       │
       ▼
  [New State]

**Ví dụ code - Actions trong Media Player:**

.. code-block:: c

  typedef enum {
      STATE_STOPPED,
      STATE_PLAYING,
      STATE_PAUSED
  } PlayerState;

  typedef struct {
      PlayerState state;
      MediaFile* current_file;
      int playback_position;
  } MediaPlayerFSM;

  // Entry Action: Called when entering PLAYING state
  void on_enter_playing(MediaPlayerFSM* fsm) {
      printf("▶ Starting playback: %s\n", fsm->current_file->filename);
      start_audio_decoder(fsm->current_file);
      start_progress_timer();
      update_ui_play_button();
  }

  // Exit Action: Called when leaving PLAYING state
  void on_exit_playing(MediaPlayerFSM* fsm) {
      printf("⏸ Stopping playback\n");
      stop_audio_decoder();
      stop_progress_timer();
      save_playback_position(fsm->playback_position);
  }

  // Transition Action: Called during STOPPED → PLAYING
  void on_transition_stop_to_play(MediaPlayerFSM* fsm) {
      log_state_change("STOPPED", "PLAYING");
      analytics_track_event("playback_started");
      check_audio_permissions();
  }

  void fsm_transition_with_actions(MediaPlayerFSM* fsm, Event event) {
      PlayerState old_state = fsm->state;
      
      // Determine new state
      PlayerState new_state = determine_next_state(fsm, event);
      
      // Execute EXIT action from old state
      if (old_state == STATE_PLAYING) {
          on_exit_playing(fsm);
      }
      
      // Execute TRANSITION action
      if (old_state == STATE_STOPPED && new_state == STATE_PLAYING) {
          on_transition_stop_to_play(fsm);
      }
      
      // Update state
      fsm->state = new_state;
      
      // Execute ENTRY action for new state
      if (new_state == STATE_PLAYING) {
          on_enter_playing(fsm);
      }
  }

.. rubric:: 6. Guards và Conditional Transitions

**Guards** là các điều kiện kiểm tra trước khi thực hiện transition. Chúng cho phép hoặc ngăn chặn việc chuyển từ state này sang state khác dựa trên điều kiện cụ thể.

Ví dụ: Chuyển từ "Authenticating" sang "Authenticated" chỉ khi password hợp lệ.

*Lưu ý: Cần phân tích sâu hơn về guards và conditional transitions trong các tài liệu chuyên sâu.*

**Sơ đồ Guards:**

::

  ┌─────────────────────────────────────────────────────┐
  │         GUARDS - CONDITIONAL TRANSITIONS             │
  └─────────────────────────────────────────────────────┘
  
  [Authenticating]
       │
       │ Login Attempt
       ▼
  ┌──────────────────┐
  │  Guard Check:    │
  │  password_valid? │
  └──────────────────┘
       │
       ├─── YES ───► [Authenticated]
       │
       └─── NO ────► [Authenticating]
                         │
                         │ (retry_count < 3)
                         └───► Allow retry
                         │
                         │ (retry_count >= 3)
                         └───► [Account Locked]

**Ví dụ code - Guards trong C:**

.. code-block:: c

  typedef enum {
      GUARD_TRUE,
      GUARD_FALSE
  } GuardResult;

  // Guard function: Check if user has permission
  GuardResult check_admin_permission(User* user) {
      if (user->role == ROLE_ADMIN) {
          return GUARD_TRUE;
      }
      return GUARD_FALSE;
  }

  // Guard function: Check if file exists
  GuardResult check_file_exists(const char* filepath) {
      if (access(filepath, F_OK) == 0) {
          return GUARD_TRUE;
      }
      return GUARD_FALSE;
  }

  // State machine with guards
  void fsm_transition_with_guards(FSM* fsm, Event event) {
      switch (fsm->current_state) {
          case STATE_IDLE:
              if (event == EVENT_OPEN_FILE) {
                  // Guard: Check file exists before transition
                  if (check_file_exists(fsm->filename) == GUARD_TRUE) {
                      fsm->current_state = STATE_READING;
                      printf("✓ File exists, transitioning to READING\n");
                  } else {
                      printf("✗ File not found, staying in IDLE\n");
                  }
              }
              break;
              
          case STATE_READING:
              if (event == EVENT_PROCESS_DATA) {
                  // Guard: Check permissions
                  if (check_admin_permission(fsm->user) == GUARD_TRUE) {
                      fsm->current_state = STATE_PROCESSING;
                      printf("✓ Permission granted, transitioning to PROCESSING\n");
                  } else {
                      printf("✗ Access denied, staying in READING\n");
                  }
              }
              break;
      }
  }

.. rubric:: 7. Common Pitfalls và Best Practices

**Các lỗi thường gặp (Pitfalls):**

- **State Explosion:** Tạo quá nhiều states mà không cần thiết, làm hệ thống phức tạp và khó bảo trì
- **Over-engineering:** Sử dụng state machine cho những bài toán đơn giản không cần thiết
- **Unclear States:** Đặt tên states không rõ ràng, khó hiểu

**Best Practices:**

- **Avoid state explosion:** Tổ chức states một cách hợp lý, sử dụng hierarchical states khi cần
- **Don't use state machines for everything:** Chỉ áp dụng khi bài toán thực sự phức tạp và có nhiều trạng thái
- **Keep states meaningful:** Đặt tên states rõ ràng, phản ánh đúng ý nghĩa
- **Make events explicit:** Định nghĩa events rõ ràng, dễ hiểu

**Ví dụ - State Explosion vs Hierarchical States:**

::

  ❌ BAD: State Explosion
  ┌──────────────────────────────────────────────────────┐
  │ MusicPlayer_Playing_Volume_Up                        │
  │ MusicPlayer_Playing_Volume_Down                      │
  │ MusicPlayer_Playing_Volume_Mute                      │
  │ MusicPlayer_Playing_Seeking_Forward                  │
  │ MusicPlayer_Playing_Seeking_Backward                 │
  │ MusicPlayer_Paused_Volume_Up                         │
  │ MusicPlayer_Paused_Volume_Down                       │
  │ ... (20+ states for simple player)                   │
  └──────────────────────────────────────────────────────┘

::

  ✅ GOOD: Hierarchical States
  ┌──────────────────────────────────────────────────────┐
  │ Main States:                                         │
  │   - STOPPED                                          │
  │   - PLAYING ──► [Sub-states: Volume, Seeking]        │
  │   - PAUSED ──► [Sub-states: Volume]                  │
  │                                                     │
  │ Total: 3 main states + nested sub-states            │
  └──────────────────────────────────────────────────────┘

.. rubric:: 8. Khi Nào Sử Dụng State Machine?

State machine đặc biệt hữu ích trong các trường hợp sau:

- **Authentication flows:** Người dùng tiến qua các trạng thái như unauthenticated → authenticating → authenticated → session expired
- **Multi-step forms/wizards:** Hướng dẫn người dùng qua các bước tuần tự với validation tại mỗi stage
- **Complex UI components:** Video players, file upload widgets, interactive tutorials với nhiều modes và behaviors
- **API orchestration:** Điều phối multiple requests, handle retries, và quản lý loading states
- **Game logic:** Characters, enemies, và game states tuân theo các quy tắc well-defined

**Ví dụ chi tiết - Authentication Flow:**

::

  ┌──────────────────────────────────────────────────────┐
  │         AUTHENTICATION STATE MACHINE                  │
  └──────────────────────────────────────────────────────┘
  
  [Unauthenticated]
       │
       │ Login Request
       ▼
  [Authenticating]
       │
       ├─── Password Valid ───► [Authenticated]
       │                            │
       │                            ├── User Activity
       │                            │       │
       │                            │       ▼
       │                            │   [Keep Alive]
       │                            │       │
       │                            │       ▼
       │                            │   [Authenticated]
       │                            │
       │                            └── Session Timeout
       │                                    │
       │                                    ▼
       │                            [Session Expired]
       │                                    │
       │                                    │ Re-login
       │                                    ▼
       │                            [Authenticating]
       │
       └── Password Invalid ──► [Unauthenticated]
                                    │
                                    │ (retry < 3)
                                    └──► Allow retry
                                    │
                                    │ (retry >= 3)
                                    └──► [Account Locked]

**Ví dụ code - Complete Order Processing System:**

.. code-block:: python

  from enum import Enum
  from dataclasses import dataclass
  from typing import Optional

  class OrderState(Enum):
      CART = "cart"
      PAYMENT_PENDING = "payment_pending"
      PAYMENT_SUCCESS = "payment_success"
      PAYMENT_FAILED = "payment_failed"
      PROCESSING = "processing"
      SHIPPED = "shipped"
      DELIVERED = "delivered"
      CANCELLED = "cancelled"

  class OrderEvent(Enum):
      ADD_TO_CART = "add_to_cart"
      CHECKOUT = "checkout"
      PAYMENT_SUCCESS = "payment_success"
      PAYMENT_FAILED = "payment_failed"
      CANCEL_ORDER = "cancel_order"
      START_PROCESSING = "start_processing"
      SHIP_ORDER = "ship_order"
      DELIVER_ORDER = "deliver_order"

  @dataclass
  class Order:
      order_id: str
      items: list
      total_amount: float
      payment_id: Optional[str] = None
      tracking_number: Optional[str] = None

  class OrderStateMachine:
      def __init__(self, order: Order):
          self.order = order
          self.state = OrderState.CART
          
      def transition(self, event: OrderEvent, **kwargs):
          print(f"\n[{self.state.value.upper()}] Event: {event.value}")
          
          # State transitions with guards
          if self.state == OrderState.CART:
              if event == OrderEvent.CHECKOUT:
                  if len(self.order.items) > 0:
                      self.state = OrderState.PAYMENT_PENDING
                      print(f"  → Proceeding to payment")
                  else:
                      print(f"  ✗ Cart is empty!")
                      
          elif self.state == OrderState.PAYMENT_PENDING:
              if event == OrderEvent.PAYMENT_SUCCESS:
                  self.order.payment_id = kwargs.get('payment_id')
                  self.state = OrderState.PAYMENT_SUCCESS
                  print(f"  → Payment successful! ID: {self.order.payment_id}")
              elif event == OrderEvent.PAYMENT_FAILED:
                  self.state = OrderState.PAYMENT_FAILED
                  print(f"  → Payment failed")
              elif event == OrderEvent.CANCEL_ORDER:
                  self.state = OrderState.CANCELLED
                  print(f"  → Order cancelled by user")
                  
          elif self.state == OrderState.PAYMENT_SUCCESS:
              if event == OrderEvent.START_PROCESSING:
                  self.state = OrderState.PROCESSING
                  print(f"  → Order is being processed")
              elif event == OrderEvent.CANCEL_ORDER:
                  # Refund logic here
                  self.state = OrderState.CANCELLED
                  print(f"  → Order cancelled, refund initiated")
                  
          elif self.state == OrderState.PROCESSING:
              if event == OrderEvent.SHIP_ORDER:
                  self.order.tracking_number = kwargs.get('tracking_number')
                  self.state = OrderState.SHIPPED
                  print(f"  → Order shipped! Tracking: {self.order.tracking_number}")
              elif event == OrderEvent.CANCEL_ORDER:
                  self.state = OrderState.CANCELLED
                  print(f"  → Order cancelled during processing")
                  
          elif self.state == OrderState.SHIPPED:
              if event == OrderEvent.DELIVER_ORDER:
                  self.state = OrderState.DELIVERED
                  print(f"  → Order delivered successfully!")
          
          print(f"  Current State: {self.state.value.upper()}")

  # Demo: Complete order flow
  if __name__ == "__main__":
      # Create an order
      order = Order(
          order_id="ORD-001",
          items=["Laptop", "Mouse", "Keyboard"],
          total_amount=1500.00
      )
      
      # Initialize state machine
      order_fsm = OrderStateMachine(order)
      
      # Simulate order flow
      order_fsm.transition(OrderEvent.CHECKOUT)
      order_fsm.transition(OrderEvent.PAYMENT_SUCCESS, payment_id="PAY-12345")
      order_fsm.transition(OrderEvent.START_PROCESSING)
      order_fsm.transition(OrderEvent.SHIP_ORDER, tracking_number="TRACK-789")
      order_fsm.transition(OrderEvent.DELIVER_ORDER)

**Output:**

::

  [CART] Event: checkout
    → Proceeding to payment
  Current State: PAYMENT_PENDING

  [PAYMENT_PENDING] Event: payment_success
    → Payment successful! ID: PAY-12345
  Current State: PAYMENT_SUCCESS

  [PAYMENT_SUCCESS] Event: start_processing
    → Order is being processed
  Current State: PROCESSING

  [PROCESSING] Event: ship_order
    → Order shipped! Tracking: TRACK-789
  Current State: SHIPPED

  [SHIPPED] Event: deliver_order
    → Order delivered successfully!
  Current State: DELIVERED

.. rubric:: Tài Liệu Tham Khảo

- `Understanding State Machines - A Developer's Guide <https://medium.com/@melekcharradi/understanding-state-machines-a-developers-guide-to-predictable-application-logic-d3df50e3e621>`_
- `State Machine Design Patterns <https://en.wikipedia.org/wiki/Finite-state_machine>`_
- `Statecharts - A Visual Formalism for Complex Systems <https://en.wikipedia.org/wiki/UML_state_machine>`_
- `State Pattern - Refactoring Guru <https://refactoring.guru/design-patterns/state>`_