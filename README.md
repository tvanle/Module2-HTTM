# BÁO CÁO MODULE: TẠO CHAT MỚI VÀ TRÒ CHUYỆN

**Sinh viên thực hiện:** Lê Văn Trọng - B22DCCN863
**Lớp học phần:** 06
**Nhóm đề tài:** 17

---

## 1. TỔNG QUAN MODULE

Module "Tạo chat mới và trò chuyện" cho phép người dùng:
- Tạo cuộc hội thoại mới với chatbot
- Gửi tin nhắn và nhận phản hồi từ AI
- Lưu trữ lịch sử chat vào cơ sở dữ liệu

---

## 2. THIẾT KẾ CƠ SỞ DỮ LIỆU

### 2.1. Bảng tblChatHistory (Lưu đoạn chat)

| Tên cột | Kiểu dữ liệu | Mô tả |
|---------|--------------|-------|
| id | BigInteger | Khóa chính, tự tăng |
| user_id | BigInteger | Khóa ngoại tới tblUser |
| title | String(500) | Tiêu đề đoạn chat |
| created_at | DateTime | Thời gian tạo |
| updated_at | DateTime | Thời gian cập nhật |

### 2.2. Bảng tblMessage (Lưu tin nhắn)

| Tên cột | Kiểu dữ liệu | Mô tả |
|---------|--------------|-------|
| id | BigInteger | Khóa chính, tự tăng |
| chat_id | BigInteger | Khóa ngoại tới tblChatHistory |
| type | Enum | "user" hoặc "assistant" |
| content | Text | Nội dung tin nhắn |
| created_at | DateTime | Thời gian tạo |

### 2.3. Sơ đồ quan hệ

```
tblUser (1) ----< (n) tblChatHistory (1) ----< (n) tblMessage (1) ---- (1) tblModel
```

---

## 3. THIẾT KẾ LỚP THỰC THỂ

### 3.1. Lớp Chat (Chat.py)

```python
class Chat(Base):
    __tablename__ = "tblChat"

    id = Column(BigInteger, primary_key=True, autoincrement=True)
    user_id = Column(BigInteger, ForeignKey("tblUser.id"), nullable=False, index=True)
    title = Column(String(500), nullable=False)
    created_at = Column(DateTime, server_default=func.now())
    updated_at = Column(DateTime, server_default=func.now(), onupdate=func.now())

    messages = relationship("Message", back_populates="chat", cascade="all, delete-orphan")
```

### 3.2. Lớp Message (Message.py)

```python
class MessageType(enum.Enum):
    user = "user"
    assistant = "assistant"

class Message(Base):
    __tablename__ = "tblMessage"

    id = Column(BigInteger, primary_key=True, autoincrement=True)
    chat_id = Column(BigInteger, ForeignKey("tblChat.id"), nullable=False, index=True)
    type = Column(SQLEnum(MessageType), nullable=False)
    content = Column(Text, nullable=False)
    created_at = Column(DateTime, server_default=func.now())

    chat = relationship("Chat", back_populates="messages")
```

---

## 4. THIẾT KẾ CÁC TẦNG XỬ LÝ

### 4.1. Tầng DAO (Data Access Object)

#### ChatDAO.py
```python
class ChatDAO:
    @staticmethod
    def create(db: Session, user_id: int, title: str) -> Chat:
        """Tạo chat mới"""

    @staticmethod
    def find_by_id(db: Session, chat_id: int) -> Chat | None:
        """Tìm chat theo ID"""

    @staticmethod
    def find_by_user(db: Session, user_id: int) -> list[Chat]:
        """Lấy danh sách chat của user"""
```

#### MessageDAO.py
```python
class MessageDAO:
    @staticmethod
    def create(db: Session, chat_id: int, msg_type: MessageType, content: str) -> Message:
        """Tạo message mới"""

    @staticmethod
    def find_by_chat(db: Session, chat_id: int) -> list[Message]:
        """Lấy messages của 1 chat"""
```

### 4.2. Tầng Service (Business Logic)

#### chatService.py
```python
class ChatService:
    @staticmethod
    def create_chat(db: Session, user_id: int, title: str):
        """Xử lý logic tạo chat mới"""

    @staticmethod
    def get_chat_list(db: Session, user_id: int):
        """Lấy danh sách chat"""

    @staticmethod
    def get_chat_messages(db: Session, chat_id: int):
        """Lấy tin nhắn của chat"""

    @staticmethod
    def send_message(db: Session, chat_id: int, content: str):
        """Gửi tin nhắn và nhận phản hồi từ bot"""
        # TODO: Tích hợp AI model
```

### 4.3. Tầng Controller (API Endpoints)

#### chat.py
```python
router = APIRouter(prefix="/api/chat", tags=["chat"])

@router.post("/new")
def new_chat(user_id: int, title: str, db: Session):
    """API tạo chat mới"""

@router.get("/list")
def get_chat_list(user_id: int, db: Session):
    """API lấy danh sách chat"""

@router.get("/messages")
def get_chat_messages(chat_id: int, db: Session):
    """API lấy tin nhắn"""

@router.post("/send")
def send_message(chat_id: int, content: str, db: Session):
    """API gửi tin nhắn"""

@router.get("/models")
def get_models():
    """API lấy danh sách AI models"""
```

---

## 5. THIẾT KẾ GIAO DIỆN (FRONTEND)

### 5.1. Cấu trúc files

```
FE/
├── js/
│   ├── chatManager.js    # Quản lý logic chat
│   ├── apiService.js      # Gọi API backend
│   ├── utils.js           # Hàm tiện ích
│   └── config.js          # Cấu hình
└── index.html             # Giao diện chính
```

### 5.2. Class ChatManager (chatManager.js)

```javascript
export class ChatManager {
    // Khởi tạo danh sách chat
    async initialize() { }

    // Render danh sách chat
    renderChatList() { }

    // Load tin nhắn của 1 chat
    async loadChat(chatId) { }

    // Tạo chat mới
    async createNewChat(firstMessage = null) { }

    // Gửi tin nhắn
    async sendMessage(text) { }
}
```

### 5.3. API Service (apiService.js)

```javascript
export const apiService = {
    async createChat(userId, title) {
        // POST /api/chat/new
    },

    async getChatList(userId) {
        // GET /api/chat/list?user_id={id}
    },

    async getChatMessages(chatId) {
        // GET /api/chat/messages?chat_id={id}
    },

    async sendMessage(chatId, content) {
        // POST /api/chat/send
    }
};
```

---

## 6. LUỒNG HOẠT ĐỘNG

### 6.1. Tạo chat mới và gửi tin nhắn đầu tiên

```
User (index.html)
  → chatManager.createNewChat(message)
    → apiService.createChat(userId, title)
      → POST /api/chat/new
        → chat.py: new_chat()
          → chatService.py: create_chat()
            → ChatDAO.py: create()
              → Lưu vào DB
            ← Trả về chat.id
          ← Trả về {ok: true, chat: {...}}
        ← Response JSON
      ← data
    → Gửi tin nhắn qua sendMessage()
      → POST /api/chat/send
        → chatService.py: send_message()
          → MessageDAO: lưu user message
          → TODO: Gọi AI model để lấy response
          → MessageDAO: lưu bot message
        ← {ok: true, bot_message: {...}}
      ← Hiển thị trên UI
```

### 6.2. Chọn chat cũ và trò chuyện

```
User click chat cũ
  → chatManager.loadChat(chatId)
    → GET /api/chat/messages?chat_id={id}
      → chatService.get_chat_messages()
        → MessageDAO.find_by_chat()
      ← {ok: true, messages: [...]}
    → Hiển thị messages lên UI

User gửi tin nhắn mới
  → chatManager.sendMessage(text)
    → POST /api/chat/send
      → chatService.send_message()
        → Lưu user message
        → Gọi AI (TODO)
        → Lưu bot message
      ← Response
    → Hiển thị lên UI
```

---

## 7. API ENDPOINTS

| Method | Endpoint | Mô tả | Request | Response |
|--------|----------|-------|---------|----------|
| POST | /api/chat/new | Tạo chat mới | `user_id`, `title` | `{ok, chat: {id, title}}` |
| GET | /api/chat/list | Lấy danh sách chat | `user_id` | `{ok, chats: [...]}` |
| GET | /api/chat/messages | Lấy tin nhắn | `chat_id` | `{ok, messages: [...]}` |
| POST | /api/chat/send | Gửi tin nhắn | `chat_id`, `content` | `{ok, user_message, bot_message}` |
| GET | /api/chat/models | Lấy danh sách models | - | `{ok, models: [...]}` |

---

## 8. CẤU TRÚC THƯ MỤC DỰ ÁN

```
BE/
├── models/
│   ├── Chat.py           # Model Chat
│   └── Message.py        # Model Message
├── dao/
│   ├── ChatDAO.py        # DAO cho Chat
│   └── MessageDAO.py     # DAO cho Message
├── services/
│   └── chatService.py    # Business logic
├── controllers/
│   └── chat.py           # API endpoints
└── main.py               # Đăng ký router

FE/
├── js/
│   ├── chatManager.js
│   ├── apiService.js
│   ├── utils.js
│   └── config.js
└── index.html
```

---

## 9. HƯỚNG PHÁT TRIỂN

### 9.1. Hiện tại
- ✅ Tạo chat mới
- ✅ Gửi/nhận tin nhắn
- ✅ Lưu trữ vào DB
- ⚠️ Bot response dùng mock data

### 9.2. Tương lai
- ❌ Tích hợp AI model (llmClient.py)
- ❌ Chọn model khác nhau
- ❌ Streaming response
- ❌ Upload file/image
- ❌ Xóa/sửa chat

---

## 10. GHI CHÚ QUAN TRỌNG

### 10.1. Khác biệt giữa thiết kế PDF và code thực tế

| Mục | Theo PDF | Theo Code | Trạng thái |
|-----|----------|-----------|------------|
| Endpoint tạo chat | `/api/chat/new` | `/api/chat/create` → sửa thành `/new` | ✅ Đã sửa |
| Tên file DAO | `chatRepo.py` | `ChatDAO.py` | ⚠️ Khác tên nhưng cùng chức năng |
| llmClient.py | Có trong PDF | Chưa implement | ❌ TODO |

### 10.2. Code đã triển khai

**Backend:**
- ✅ Models: Chat.py, Message.py
- ✅ DAO: ChatDAO.py, MessageDAO.py
- ✅ Service: chatService.py
- ✅ Controller: chat.py
- ⚠️ llmClient.py: Chưa có (dùng mock response)

**Frontend:**
- ✅ chatManager.js: Quản lý chat
- ✅ apiService.js: Gọi API
- ✅ Giao diện index.html
- ✅ Tích hợp hoàn chỉnh

---

## KẾT LUẬN

Module "Tạo chat mới và trò chuyện" đã được triển khai thành công với đầy đủ các tầng:
- **Database**: Models và migrations
- **DAO Layer**: Truy xuất dữ liệu
- **Service Layer**: Logic nghiệp vụ
- **Controller Layer**: API endpoints
- **Frontend**: Giao diện và tích hợp API

**Điểm còn thiếu:** Tích hợp AI model thật (hiện dùng mock response).

---

*Báo cáo được tạo dựa trên code thực tế của dự án Chatbot17*
