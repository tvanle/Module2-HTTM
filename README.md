# BÁO CÁO THIẾT KẾ MODULE CHAT - PHÍA CLIENT

## 1. TỔNG QUAN HỆ THỐNG

### 1.1. Mô tả
Module Chat là phần frontend của hệ thống AMI RAG-Powered Chatbot, cho phép người dùng tương tác với AI Assistant thông qua giao diện web. Module này xử lý việc gửi tin nhắn, hiển thị phản hồi, quản lý phiên chat và cấu hình các tham số hội thoại.

### 1.2. Công nghệ sử dụng
- **Frontend Framework**: React 18 + TypeScript
- **Build Tool**: Vite
- **State Management**: Zustand
- **HTTP Client**: Axios
- **UI Components**: Custom Components + Material-UI
- **Styling**: CSS Modules

### 1.3. Kiến trúc tổng quan
```
┌─────────────────────────────────────────────────────────┐
│                    Frontend Layer                        │
│  ┌──────────┐  ┌──────────┐  ┌─────────────────────┐  │
│  │   Pages  │  │Components│  │  State Management    │  │
│  │          │  │          │  │      (Zustand)       │  │
│  └─────┬────┘  └────┬─────┘  └──────────┬──────────┘  │
│        │            │                    │              │
│        └────────────┴────────────────────┘              │
│                     │                                    │
│            ┌────────▼────────┐                          │
│            │   API Client    │                          │
│            │    (Axios)      │                          │
│            └────────┬────────┘                          │
└─────────────────────┼──────────────────────────────────┘
                      │ HTTP/REST
                      ▼
          ┌───────────────────────┐
          │   Backend API         │
          │   (FastAPI)           │
          │   Port: 6008          │
          └───────────────────────┘
```

---

## 2. THIẾT KẾ CƠ SỞ DỮ LIỆU (MongoDB Schema)

### 2.1. Collection: chat_sessions
```typescript
interface ChatSessionInDB {
  _id: ObjectId;                    // ID phiên chat (được map thành id)
  user_id: string;                  // ID người dùng sở hữu
  title: string;                    // Tiêu đề phiên chat (mặc định: "New Conversation")
  summary?: string;                 // Tóm tắt nội dung (tối đa 2000 ký tự)
  message_count: number;            // Số lượng tin nhắn trong phiên
  last_message_at?: Date;           // Thời gian tin nhắn cuối cùng
  created_at: Date;                 // Thời gian tạo
  updated_at: Date;                 // Thời gian cập nhật
  is_archived: boolean;             // Trạng thái lưu trữ
  is_deleted: boolean;              // Trạng thái xóa mềm
  metadata: Record<string, any>;    // Metadata mở rộng
  tags: string[];                   // Tags phân loại
}
```

**Indexes:**
- `user_id` (ascending) - Tìm kiếm phiên theo người dùng
- `created_at` (descending) - Sắp xếp theo thời gian tạo
- `is_deleted` (ascending) - Filter phiên đã xóa
- `{ user_id: 1, is_deleted: 1, created_at: -1 }` - Composite index

### 2.2. Collection: chat_messages
```typescript
interface ChatMessageInDB {
  _id: ObjectId;                    // ID tin nhắn (được map thành id)
  session_id: string;               // ID phiên chat (foreign key)
  role: 'system' | 'user' | 'assistant';  // Vai trò người gửi
  content: string;                  // Nội dung tin nhắn
  attachments: FileAttachment[];    // Danh sách file đính kèm
  metadata: Record<string, any>;    // Metadata (thinking_mode, rag_enabled, etc.)
  created_at: Date;                 // Thời gian tạo
  edited_at?: Date;                 // Thời gian chỉnh sửa
  is_deleted: boolean;              // Trạng thái xóa mềm
}

interface FileAttachment {
  file_id: string;                  // ID file trong MinIO
  type: string;                     // image | document | audio | video
  url: string;                      // URL truy cập file
  thumbnail_url?: string;           // URL thumbnail (nếu là ảnh)
  filename: string;                 // Tên file gốc
  size: number;                     // Kích thước (bytes)
  mime_type: string;                // MIME type
  width?: number;                   // Chiều rộng (ảnh)
  height?: number;                  // Chiều cao (ảnh)
  generated: boolean;               // File được tạo bởi AI
  generation_prompt?: string;       // Prompt tạo ảnh
  vision_analysis?: Record<string, any>; // Phân tích Vision AI
}
```

**Indexes:**
- `session_id` (ascending) - Tìm tin nhắn theo phiên
- `created_at` (ascending) - Sắp xếp theo thời gian
- `{ session_id: 1, is_deleted: 1, created_at: 1 }` - Composite index

### 2.3. Collection: users (liên quan)
```typescript
interface UserInDB {
  _id: ObjectId;                    // ID người dùng
  username: string;                 // Tên đăng nhập (unique)
  email: string;                    // Email (unique)
  full_name?: string;               // Tên đầy đủ
  hashed_password: string;          // Mật khẩu đã hash
  role: 'admin' | 'user';           // Vai trò
  is_active: boolean;               // Trạng thái hoạt động
  created_at: Date;                 // Thời gian tạo
  updated_at: Date;                 // Thời gian cập nhật
  last_login?: Date;                // Lần đăng nhập cuối
}
```

**Indexes:**
- `username` (unique) - Tìm kiếm người dùng
- `email` (unique) - Tìm kiếm theo email

---

## 3. THIẾT KẾ LỚP THỰC THỂ (Entity Classes)

### 3.1. Domain Models (Frontend)

```typescript
// ============ Chat Session Entity ============
export interface ChatSession {
  id: string;                       // ID phiên chat
  user_id: string;                  // ID người dùng
  title: string;                    // Tiêu đề
  summary?: string;                 // Tóm tắt
  message_count: number;            // Số lượng tin nhắn
  created_at: string;               // ISO datetime
  updated_at: string;               // ISO datetime
  last_message_at: string;          // ISO datetime
  is_archived: boolean;             // Đã lưu trữ
  is_deleted: boolean;              // Đã xóa
}

// ============ Chat Message Entity ============
export interface ChatMessage {
  id: string;                       // ID tin nhắn
  session_id: string;               // ID phiên chat
  role: 'user' | 'assistant';       // Vai trò
  content: string;                  // Nội dung
  metadata?: MessageMetadata;       // Metadata
  created_at: string;               // ISO datetime
  edited_at?: string;               // ISO datetime
  is_deleted: boolean;              // Đã xóa
}

export interface MessageMetadata {
  thinking_mode?: 'fast' | 'balance' | 'thinking';
  rag_enabled?: boolean;            // RAG có được bật không
  web_search_enabled?: boolean;     // Web search có được bật không
  sources?: DocumentSource[];       // Nguồn tài liệu RAG
  web_sources?: WebSource[];        // Nguồn web search
  conversation_turns?: number;      // Số lượt hội thoại
}

export interface DocumentSource {
  content: string;                  // Nội dung trích xuất
  metadata: {
    file_name?: string;             // Tên file nguồn
    page?: number;                  // Số trang
    chunk_index?: number;           // Chỉ số chunk
    collection?: string;            // Collection
  };
  score: number;                    // Điểm similarity
}

export interface WebSource {
  url: string;                      // URL nguồn
  title: string;                    // Tiêu đề trang
  content: string;                  // Nội dung trích xuất
  snippet: string;                  // Đoạn trích
}

// ============ Request/Response Models ============
export interface Message {
  role: 'system' | 'user' | 'assistant';
  content: string;
}

export interface ChatRequest {
  messages: Message[];              // Lịch sử hội thoại
  thinking_mode?: 'fast' | 'balance' | 'thinking';
  system_prompt?: string;           // System prompt tùy chỉnh
  rag_config?: RAGConfig;           // Cấu hình RAG
  web_search_config?: WebSearchConfig; // Cấu hình web search
  generation_config?: GenerationConfig; // Cấu hình generation
  collection?: string;              // Collection để search
  stream?: boolean;                 // Streaming mode
  session_id?: string;              // ID phiên chat (để lưu lịch sử)
  auto_generate_title?: boolean;    // Tự động tạo tiêu đề
}

export interface RAGConfig {
  enabled: boolean;                 // Bật/tắt RAG
  top_k: number;                    // Số lượng documents trả về (mặc định: 5)
  similarity_threshold: number;     // Ngưỡng similarity (0.0-1.0)
  include_sources: boolean;         // Trả về nguồn tài liệu
  metadata_filter?: Record<string, unknown>; // Filter metadata
}

export interface WebSearchConfig {
  enabled: boolean;                 // Bật/tắt web search
  max_results: number;              // Số kết quả tối đa (mặc định: 5)
  timeout: number;                  // Timeout (ms)
}

export interface GenerationConfig {
  temperature: number;              // 0.0-2.0 (mặc định: 0.7)
  max_tokens?: number;              // Số tokens tối đa
  top_p: number;                    // Nucleus sampling (mặc định: 1.0)
  frequency_penalty: number;        // 0.0-2.0 (mặc định: 0)
  presence_penalty: number;         // 0.0-2.0 (mặc định: 0)
}

export interface ChatResponse {
  message: Message;                 // Phản hồi từ AI
  sources?: DocumentSource[];       // Nguồn RAG
  web_sources?: WebSource[];        // Nguồn web
  metadata: ResponseMetadata;       // Metadata phản hồi
  session_id?: string;              // ID phiên chat (nếu đã lưu)
}

export interface ResponseMetadata {
  thinking_mode: string;            // Thinking mode đã dùng
  collection: string;               // Collection đã search
  conversation_turns: number;       // Số lượt hội thoại
  rag_enabled: boolean;             // RAG đã bật
  web_search_enabled: boolean;      // Web search đã bật
}
```

### 3.2. State Management Models

```typescript
// ============ Chat Store State ============
export interface ChatState {
  // Session Management
  currentSession: ChatSession | null;      // Phiên chat hiện tại
  sessions: ChatSession[];                 // Danh sách phiên chat

  // Message Management
  currentMessages: ChatMessage[];          // Tin nhắn của phiên hiện tại

  // UI State
  loading: boolean;                        // Đang load
  error: string | null;                    // Thông báo lỗi
  isUploadingFile: boolean;                // Đang upload file

  // Configuration
  config: ChatConfig;                      // Cấu hình chat

  // Actions
  setCurrentSession: (session: ChatSession | null) => void;
  setSessions: (sessions: ChatSession[]) => void;
  addMessage: (message: ChatMessage) => void;
  setCurrentMessages: (messages: ChatMessage[]) => void;
  setLoading: (loading: boolean) => void;
  setError: (error: string | null) => void;
  updateConfig: (config: Partial<ChatConfig>) => void;
  setIsUploadingFile: (loading: boolean) => void;
  clearState: () => void;
}

export interface ChatConfig {
  thinkingMode: 'fast' | 'balance' | 'thinking';  // Mặc định: 'balance'
  enableRAG: boolean;                      // Mặc định: true
  enableWebSearch: boolean;                // Mặc định: false
  temperature: number;                     // Mặc định: 0.7
  maxTokens?: number;                      // Tùy chọn
  topK: number;                            // Mặc định: 5
  similarityThreshold: number;             // Mặc định: 0.0
  collection: string;                      // Mặc định: 'default'
}
```

---

## 4. BIỂU ĐỒ LỚP CHI TIẾT (Class Diagram)

```mermaid
classDiagram
    %% ========== API Client Layer ==========
    class APIClient {
        -client: AxiosInstance
        +generateChat(request: ChatRequest): Promise~ChatResponse~
        +createSession(title?: string): Promise~ChatSession~
        +listSessions(skip, limit, isArchived?, search?): Promise~SessionList~
        +getSession(sessionId, includeMessages): Promise~ChatSessionWithMessages~
        +updateSession(sessionId, updates): Promise~ChatSession~
        +deleteSession(sessionId): Promise~void~
        +archiveSession(sessionId): Promise~void~
        +restoreSession(sessionId): Promise~void~
        +addMessage(sessionId, role, content, metadata?): Promise~ChatMessage~
        +listMessages(sessionId, skip, limit): Promise~MessageList~
        +summarizeSession(sessionId): Promise~SummaryResponse~
        +uploadFile(file, collection): Promise~UploadResponse~
    }

    %% ========== State Management Layer ==========
    class ChatStore {
        +currentSession: ChatSession | null
        +sessions: ChatSession[]
        +currentMessages: ChatMessage[]
        +loading: boolean
        +error: string | null
        +isUploadingFile: boolean
        +config: ChatConfig
        +setCurrentSession(session): void
        +setSessions(sessions): void
        +addMessage(message): void
        +setCurrentMessages(messages): void
        +setLoading(loading): void
        +setError(error): void
        +updateConfig(config): void
        +setIsUploadingFile(loading): void
        +clearState(): void
    }

    %% ========== UI Component Layer ==========
    class ChatPage {
        -showSettings: boolean
        -showSidebar: boolean
        -messagesEndRef: RefObject
        +loadSessions(): Promise~void~
        +loadSession(sessionId): Promise~void~
        +handleNewChat(): Promise~void~
        +handleSendMessage(message, files): Promise~void~
    }

    class ChatSidebar {
        +isOpen: boolean
        +onNewChat: Function
        +onLoadSession: Function
        +render(): ReactElement
    }

    class ChatHeader {
        +onToggleSidebar: Function
        +onToggleSettings: Function
        +render(): ReactElement
    }

    class MessageList {
        +messages: ChatMessage[]
        +isLoading: boolean
        +render(): ReactElement
    }

    class MessageInput {
        +onSendMessage: Function
        +disabled: boolean
        +render(): ReactElement
    }

    class SettingsPanel {
        +onClose: Function
        +render(): ReactElement
    }

    class MessageBubble {
        +message: ChatMessage
        +render(): ReactElement
    }

    %% ========== Domain Models ==========
    class ChatSession {
        +id: string
        +user_id: string
        +title: string
        +summary?: string
        +message_count: number
        +created_at: string
        +updated_at: string
        +last_message_at: string
        +is_archived: boolean
        +is_deleted: boolean
    }

    class ChatMessage {
        +id: string
        +session_id: string
        +role: MessageRole
        +content: string
        +metadata?: MessageMetadata
        +created_at: string
        +edited_at?: string
        +is_deleted: boolean
    }

    class ChatRequest {
        +messages: Message[]
        +thinking_mode?: ThinkingMode
        +system_prompt?: string
        +rag_config?: RAGConfig
        +web_search_config?: WebSearchConfig
        +generation_config?: GenerationConfig
        +collection?: string
        +stream?: boolean
        +session_id?: string
        +auto_generate_title?: boolean
    }

    class ChatResponse {
        +message: Message
        +sources?: DocumentSource[]
        +web_sources?: WebSource[]
        +metadata: ResponseMetadata
        +session_id?: string
    }

    class ChatConfig {
        +thinkingMode: ThinkingMode
        +enableRAG: boolean
        +enableWebSearch: boolean
        +temperature: number
        +maxTokens?: number
        +topK: number
        +similarityThreshold: number
        +collection: string
    }

    class RAGConfig {
        +enabled: boolean
        +top_k: number
        +similarity_threshold: number
        +include_sources: boolean
        +metadata_filter?: Object
    }

    class WebSearchConfig {
        +enabled: boolean
        +max_results: number
        +timeout: number
    }

    class GenerationConfig {
        +temperature: number
        +max_tokens?: number
        +top_p: number
        +frequency_penalty: number
        +presence_penalty: number
    }

    %% ========== Relationships ==========
    ChatPage --> ChatStore : uses
    ChatPage --> APIClient : uses
    ChatPage --> ChatSidebar : contains
    ChatPage --> ChatHeader : contains
    ChatPage --> MessageList : contains
    ChatPage --> MessageInput : contains
    ChatPage --> SettingsPanel : contains

    MessageList --> MessageBubble : contains

    ChatStore --> ChatSession : manages
    ChatStore --> ChatMessage : manages
    ChatStore --> ChatConfig : stores

    APIClient --> ChatRequest : sends
    APIClient --> ChatResponse : receives
    APIClient --> ChatSession : manages
    APIClient --> ChatMessage : manages

    ChatRequest --> RAGConfig : contains
    ChatRequest --> WebSearchConfig : contains
    ChatRequest --> GenerationConfig : contains

    ChatResponse --> ChatMessage : contains
```

---

## 5. BIỂU ĐỒ HOẠT ĐỘNG (Activity Diagrams)

### 5.1. Luồng gửi tin nhắn chat

```mermaid
flowchart TD
    Start([Người dùng nhập tin nhắn]) --> CheckInput{Kiểm tra input}

    CheckInput -->|Rỗng và không có file| End([Kết thúc])
    CheckInput -->|Có nội dung hoặc file| SetLoading[Bật trạng thái loading]

    SetLoading --> CheckSession{Có session hiện tại?}

    CheckSession -->|Không| CreateSession[Tạo session mới<br/>POST /chat-history/sessions]
    CreateSession --> SetSession[Lưu session vào state]

    CheckSession -->|Có| CheckFiles{Có files đính kèm?}
    SetSession --> CheckFiles

    CheckFiles -->|Có| UploadFiles[Upload files<br/>POST /vectordb/upload/file]
    CheckFiles -->|Không| AddUserMsg[Thêm tin nhắn user vào UI]

    UploadFiles --> AddUserMsg
    AddUserMsg --> SaveUserMsg[Lưu tin nhắn user vào DB<br/>POST /chat-history/sessions/:id/messages]

    SaveUserMsg --> PrepareRequest[Chuẩn bị ChatRequest]

    PrepareRequest --> BuildRequest[Xây dựng request object:<br/>- messages: conversation history<br/>- thinking_mode: từ config<br/>- rag_config: từ config<br/>- web_search_config: từ config<br/>- generation_config: từ config<br/>- session_id: current session<br/>- auto_generate_title: true]

    BuildRequest --> SendRequest[Gửi request<br/>POST /generate/chat]

    SendRequest --> WaitResponse[Chờ phản hồi từ server]

    WaitResponse --> CheckError{Có lỗi?}

    CheckError -->|Có| ShowError[Hiển thị thông báo lỗi<br/>setError]
    CheckError -->|Không| ParseResponse[Parse ChatResponse:<br/>- message.content<br/>- sources<br/>- web_sources<br/>- metadata<br/>- session_id]

    ShowError --> TurnOffLoading[Tắt trạng thái loading]

    ParseResponse --> AddAssistantMsg[Thêm tin nhắn assistant vào UI]
    AddAssistantMsg --> SaveAssistantMsg[Lưu tin nhắn assistant vào DB<br/>POST /chat-history/sessions/:id/messages]

    SaveAssistantMsg --> UpdateMessages[Cập nhật currentMessages trong state]
    UpdateMessages --> TurnOffLoading

    TurnOffLoading --> AutoScroll[Tự động scroll xuống cuối]
    AutoScroll --> End

    style Start fill:#90EE90
    style End fill:#FFB6C1
    style CheckError fill:#FFE4B5
    style ShowError fill:#FF6B6B
    style SendRequest fill:#87CEEB
    style SaveUserMsg fill:#DDA0DD
    style SaveAssistantMsg fill:#DDA0DD
```

### 5.2. Luồng load phiên chat

```mermaid
flowchart TD
    Start([Người dùng click vào session]) --> SetLoading[Bật trạng thái loading]

    SetLoading --> SendRequest[Gửi request lấy session<br/>GET /chat-history/sessions/:id<br/>?include_messages=true]

    SendRequest --> WaitResponse[Chờ phản hồi từ server]

    WaitResponse --> CheckError{Có lỗi?}

    CheckError -->|Có| ShowError[Hiển thị thông báo lỗi]
    CheckError -->|Không| ParseResponse[Parse response:<br/>- session metadata<br/>- messages array]

    ShowError --> TurnOffLoading[Tắt trạng thái loading]

    ParseResponse --> SetSession[setCurrentSession]
    SetSession --> SetMessages[setCurrentMessages]
    SetMessages --> TurnOffLoading

    TurnOffLoading --> ScrollToBottom[Scroll xuống tin nhắn cuối]
    ScrollToBottom --> End([Kết thúc])

    style Start fill:#90EE90
    style End fill:#FFB6C1
    style CheckError fill:#FFE4B5
    style ShowError fill:#FF6B6B
    style SendRequest fill:#87CEEB
```

### 5.3. Luồng tạo phiên chat mới

```mermaid
flowchart TD
    Start([Người dùng click 'New Chat']) --> SetLoading[Bật trạng thái loading]

    SetLoading --> SendRequest[Gửi request tạo session<br/>POST /chat-history/sessions<br/>body: title: 'New Chat']

    SendRequest --> WaitResponse[Chờ phản hồi từ server]

    WaitResponse --> CheckError{Có lỗi?}

    CheckError -->|Có| ShowError[Hiển thị thông báo lỗi]
    CheckError -->|Không| ParseResponse[Parse ChatSession response]

    ShowError --> TurnOffLoading[Tắt trạng thái loading]

    ParseResponse --> SetSession[setCurrentSession<br/>với session mới]
    SetSession --> ClearMessages[setCurrentMessages<br/>với mảng rỗng]
    ClearMessages --> TurnOffLoading

    TurnOffLoading --> End([Kết thúc])

    style Start fill:#90EE90
    style End fill:#FFB6C1
    style CheckError fill:#FFE4B5
    style ShowError fill:#FF6B6B
    style SendRequest fill:#87CEEB
```

### 5.4. Luồng cập nhật cấu hình chat

```mermaid
flowchart TD
    Start([Người dùng mở Settings Panel]) --> ShowPanel[Hiển thị SettingsPanel]

    ShowPanel --> UserChange{Người dùng thay đổi cấu hình}

    UserChange -->|Thinking Mode| UpdateThinking[updateConfig<br/>thinkingMode: value]
    UserChange -->|Enable RAG| UpdateRAG[updateConfig<br/>enableRAG: value]
    UserChange -->|Enable Web Search| UpdateWeb[updateConfig<br/>enableWebSearch: value]
    UserChange -->|Temperature| UpdateTemp[updateConfig<br/>temperature: value]
    UserChange -->|Top K| UpdateTopK[updateConfig<br/>topK: value]
    UserChange -->|Similarity Threshold| UpdateSim[updateConfig<br/>similarityThreshold: value]
    UserChange -->|Collection| UpdateCol[updateConfig<br/>collection: value]

    UpdateThinking --> MergeConfig[Merge với config hiện tại<br/>trong Zustand store]
    UpdateRAG --> MergeConfig
    UpdateWeb --> MergeConfig
    UpdateTemp --> MergeConfig
    UpdateTopK --> MergeConfig
    UpdateSim --> MergeConfig
    UpdateCol --> MergeConfig

    MergeConfig --> UpdateState[Cập nhật state.config]
    UpdateState --> ReRender[Re-render UI]
    ReRender --> UserChange

    UserChange -->|Đóng panel| ClosePanel[Đóng SettingsPanel]
    ClosePanel --> End([Kết thúc])

    style Start fill:#90EE90
    style End fill:#FFB6C1
    style MergeConfig fill:#DDA0DD
```

---

## 6. BIỂU ĐỒ TUẦN TỰ (Sequence Diagrams)

### 6.1. Tuần tự gửi tin nhắn chat

```mermaid
sequenceDiagram
    actor User as Người dùng
    participant UI as ChatPage
    participant Store as ChatStore
    participant API as APIClient
    participant Backend as FastAPI Server
    participant DB as MongoDB

    User->>UI: Nhập tin nhắn và nhấn Send
    UI->>UI: handleSendMessage(message, files)

    alt Không có session
        UI->>API: createSession()
        API->>Backend: POST /api/v1/chat-history/sessions
        Backend->>DB: INSERT chat_sessions
        DB-->>Backend: session_id
        Backend-->>API: ChatSession
        API-->>UI: ChatSession
        UI->>Store: setCurrentSession(session)
    end

    alt Có files đính kèm
        loop Mỗi file
            UI->>API: uploadFile(file, collection)
            API->>Backend: POST /api/v1/vectordb/upload/file
            Backend->>Backend: Process & vectorize file
            Backend->>DB: Save metadata
            Backend-->>API: UploadResponse
            API-->>UI: UploadResponse
        end
    end

    UI->>Store: addMessage(userMessage)
    UI->>API: addMessage(session_id, 'user', message)
    API->>Backend: POST /api/v1/chat-history/sessions/:id/messages
    Backend->>DB: INSERT chat_messages
    DB-->>Backend: message_id
    Backend-->>API: ChatMessage
    API-->>UI: ChatMessage

    UI->>UI: Chuẩn bị ChatRequest
    Note over UI: messages, thinking_mode,<br/>rag_config, web_search_config,<br/>generation_config, session_id

    UI->>API: generateChat(request)
    API->>Backend: POST /api/v1/generate/chat

    Backend->>DB: Load conversation history<br/>(last 10 messages)
    DB-->>Backend: messages[]

    alt RAG enabled
        Backend->>Backend: Query Qdrant vector DB
        Backend->>Backend: Retrieve relevant documents
    end

    alt Web Search enabled
        Backend->>Backend: Query web via Firecrawl API
        Backend->>Backend: Extract web sources
    end

    Backend->>Backend: Generate response via OpenAI LLM<br/>(with conversation context)

    Backend->>DB: Save user message
    Backend->>DB: Save assistant message

    alt auto_generate_title == true
        Backend->>Backend: Check if should generate title<br/>(after 2+ messages)
        alt Should generate
            Backend->>Backend: Generate title via LLM
            Backend->>DB: UPDATE chat_sessions SET title
        end
    end

    Backend-->>API: ChatResponse<br/>(message, sources, metadata, session_id)
    API-->>UI: ChatResponse

    UI->>Store: addMessage(assistantMessage)
    UI->>API: addMessage(session_id, 'assistant', response)
    API->>Backend: POST /api/v1/chat-history/sessions/:id/messages
    Backend->>DB: INSERT chat_messages
    DB-->>Backend: message_id
    Backend-->>API: ChatMessage
    API-->>UI: ChatMessage

    UI->>Store: setCurrentMessages([...messages, userMsg, assistantMsg])
    UI->>UI: Scroll to bottom
    UI-->>User: Hiển thị phản hồi
```

### 6.2. Tuần tự load phiên chat

```mermaid
sequenceDiagram
    actor User as Người dùng
    participant Sidebar as ChatSidebar
    participant UI as ChatPage
    participant Store as ChatStore
    participant API as APIClient
    participant Backend as FastAPI Server
    participant DB as MongoDB

    User->>Sidebar: Click vào session trong danh sách
    Sidebar->>UI: onLoadSession(sessionId)
    UI->>Store: setLoading(true)

    UI->>API: getSession(sessionId, includeMessages=true)
    API->>Backend: GET /api/v1/chat-history/sessions/:id<br/>?include_messages=true

    Backend->>DB: SELECT * FROM chat_sessions<br/>WHERE _id = :id AND user_id = :user_id
    DB-->>Backend: session_doc

    Backend->>DB: SELECT * FROM chat_messages<br/>WHERE session_id = :id<br/>AND is_deleted = false<br/>ORDER BY created_at ASC
    DB-->>Backend: messages[]

    Backend-->>API: ChatSessionWithMessages<br/>(session + messages)
    API-->>UI: ChatSessionWithMessages

    UI->>Store: setCurrentSession(session)
    UI->>Store: setCurrentMessages(messages)
    UI->>Store: setLoading(false)
    UI->>UI: Scroll to bottom
    UI-->>User: Hiển thị tin nhắn
```

### 6.3. Tuần tự khởi tạo ứng dụng

```mermaid
sequenceDiagram
    actor User as Người dùng
    participant App as React App
    participant UI as ChatPage
    participant Store as ChatStore
    participant API as APIClient
    participant Backend as FastAPI Server
    participant DB as MongoDB

    User->>App: Truy cập ứng dụng
    App->>UI: Mount ChatPage component

    UI->>UI: useEffect(() => loadSessions(), [])

    UI->>API: listSessions(skip=0, limit=50)
    API->>Backend: GET /api/v1/chat-history/sessions<br/>?skip=0&limit=50

    Backend->>DB: SELECT * FROM chat_sessions<br/>WHERE user_id = :user_id<br/>AND is_deleted = false<br/>ORDER BY last_message_at DESC<br/>LIMIT 50
    DB-->>Backend: sessions[]

    Backend-->>API: ChatSessionListResponse<br/>(sessions, total, skip, limit)
    API-->>UI: SessionList

    alt Có sessions
        UI->>UI: loadSession(sessions[0].id)
        Note over UI: Load phiên chat đầu tiên<br/>(xem sequence diagram 6.2)
    end

    UI-->>User: Hiển thị giao diện chat
```

### 6.4. Tuần tự xóa phiên chat

```mermaid
sequenceDiagram
    actor User as Người dùng
    participant Sidebar as ChatSidebar
    participant UI as ChatPage
    participant API as APIClient
    participant Backend as FastAPI Server
    participant DB as MongoDB

    User->>Sidebar: Click nút xóa session
    Sidebar->>UI: onDeleteSession(sessionId)

    UI->>API: deleteSession(sessionId)
    API->>Backend: DELETE /api/v1/chat-history/sessions/:id

    Backend->>DB: UPDATE chat_sessions<br/>SET is_deleted = true, updated_at = NOW()<br/>WHERE _id = :id AND user_id = :user_id
    DB-->>Backend: modified_count

    Backend->>DB: UPDATE chat_messages<br/>SET is_deleted = true<br/>WHERE session_id = :id
    DB-->>Backend: modified_count

    Backend-->>API: {message: "Session deleted successfully"}
    API-->>UI: Success

    UI->>UI: Reload sessions list
    UI-->>User: Cập nhật UI
```

---

## 7. LUỒNG DỮ LIỆU CHI TIẾT

### 7.1. Request Flow: Gửi tin nhắn

**Bước 1: User Input → UI Component**
```typescript
// MessageInput.tsx
const handleSubmit = () => {
  onSendMessage(inputText, attachedFiles);
};
```

**Bước 2: UI Component → ChatPage Handler**
```typescript
// Chat.tsx
const handleSendMessage = async (message: string, files: File[] = []) => {
  if (!message.trim() && files.length === 0) return;

  setLoading(true);

  // Tạo hoặc lấy session
  let sessionId = currentSession?.id;
  if (!sessionId) {
    const session = await apiClient.createSession();
    setCurrentSession(session);
    sessionId = session.id;
  }

  // Upload files nếu có
  if (files.length > 0) {
    for (const file of files) {
      await apiClient.uploadFile(file, config.collection);
    }
  }

  // Thêm tin nhắn user vào UI
  const userMsg = await apiClient.addMessage(sessionId, 'user', message);
  setCurrentMessages([...currentMessages, userMsg]);

  // Tạo ChatRequest từ config
  const request: ChatRequest = {
    messages: [...currentMessages, { role: 'user', content: message }]
      .map(m => ({ role: m.role, content: m.content })),
    thinking_mode: config.thinkingMode,
    rag_config: {
      enabled: config.enableRAG,
      top_k: config.topK,
      similarity_threshold: config.similarityThreshold,
      include_sources: true,
    },
    web_search_config: {
      enabled: config.enableWebSearch,
      max_results: 5,
      timeout: 30000,
    },
    generation_config: {
      temperature: config.temperature,
      max_tokens: config.maxTokens,
      top_p: 1.0,
      frequency_penalty: 0,
      presence_penalty: 0,
    },
    collection: config.collection,
    session_id: sessionId,
    auto_generate_title: true,
  };

  // Gọi API
  const response = await apiClient.generateChat(request);

  // Thêm phản hồi vào UI
  const assistantMsg = await apiClient.addMessage(
    sessionId,
    'assistant',
    response.message.content,
    response.metadata
  );
  setCurrentMessages([...currentMessages, userMsg, assistantMsg]);

  setLoading(false);
};
```

**Bước 3: API Client → HTTP Request**
```typescript
// client.ts
async generateChat(request: ChatRequest): Promise<ChatResponse> {
  const response = await this.client.post<ChatResponse>(
    '/generate/chat',
    request
  );
  return response.data;
}
```

**Bước 4: HTTP Request → Backend API**
```
POST /api/v1/generate/chat
Authorization: Bearer <JWT_TOKEN>
Content-Type: application/json

{
  "messages": [
    {"role": "user", "content": "Xin chào"},
    {"role": "assistant", "content": "Chào bạn! Tôi có thể giúp gì?"},
    {"role": "user", "content": "PTIT là gì?"}
  ],
  "thinking_mode": "balance",
  "rag_config": {
    "enabled": true,
    "top_k": 5,
    "similarity_threshold": 0.0,
    "include_sources": true
  },
  "web_search_config": {
    "enabled": false,
    "max_results": 5,
    "timeout": 30000
  },
  "generation_config": {
    "temperature": 0.7,
    "max_tokens": 2000,
    "top_p": 1.0,
    "frequency_penalty": 0,
    "presence_penalty": 0
  },
  "collection": "default",
  "session_id": "674bf12345abcdef67890123",
  "auto_generate_title": true
}
```

**Bước 5: Backend Processing**
- Load 10 tin nhắn gần nhất từ MongoDB
- Query Qdrant vector DB (nếu RAG enabled)
- Tạo context từ documents
- Gọi OpenAI API với conversation history
- Lưu tin nhắn vào MongoDB
- Auto-generate title (nếu cần)

**Bước 6: Backend → HTTP Response**
```json
{
  "message": {
    "role": "assistant",
    "content": "PTIT là Học viện Công nghệ Bưu chính Viễn thông..."
  },
  "sources": [
    {
      "content": "Học viện Công nghệ Bưu chính Viễn thông...",
      "metadata": {
        "file_name": "gioi-thieu-ptit.pdf",
        "page": 1,
        "chunk_index": 0,
        "collection": "default"
      },
      "score": 0.92
    }
  ],
  "web_sources": null,
  "metadata": {
    "thinking_mode": "balance",
    "collection": "default",
    "conversation_turns": 2,
    "rag_enabled": true,
    "web_search_enabled": false
  },
  "session_id": "674bf12345abcdef67890123"
}
```

**Bước 7: Response → UI Update**
```typescript
// Chat.tsx
const assistantMsg = await apiClient.addMessage(
  sessionId,
  'assistant',
  response.message.content,
  response.metadata
);

setCurrentMessages([...currentMessages, userMsg, assistantMsg]);
```

### 7.2. Response Flow: Hiển thị tin nhắn

**Bước 1: State Update**
```typescript
// Zustand store
setCurrentMessages([
  ...state.currentMessages,
  newUserMessage,
  newAssistantMessage
]);
```

**Bước 2: Component Re-render**
```typescript
// Chat.tsx
<MessageList messages={currentMessages} isLoading={loading} />
```

**Bước 3: Render Message Bubbles**
```typescript
// MessageList.tsx
{messages.map((msg) => (
  <MessageBubble key={msg.id} message={msg} />
))}
```

**Bước 4: Display Content**
```typescript
// MessageBubble.tsx
<div className={`message-bubble ${message.role}`}>
  <div className="message-content">
    {message.content}
  </div>
  {message.metadata?.sources && (
    <div className="sources">
      {/* Hiển thị nguồn tài liệu */}
    </div>
  )}
</div>
```

---

## 8. XỬ LÝ LỖI VÀ VALIDATION

### 8.1. Client-side Validation

```typescript
// Validation rules
const validateMessage = (message: string, files: File[]): boolean => {
  // Kiểm tra tin nhắn rỗng
  if (!message.trim() && files.length === 0) {
    return false;
  }

  // Kiểm tra độ dài tin nhắn
  if (message.length > 10000) {
    setError('Tin nhắn quá dài (tối đa 10,000 ký tự)');
    return false;
  }

  // Kiểm tra kích thước file
  const MAX_FILE_SIZE = 10 * 1024 * 1024; // 10MB
  for (const file of files) {
    if (file.size > MAX_FILE_SIZE) {
      setError(`File ${file.name} quá lớn (tối đa 10MB)`);
      return false;
    }
  }

  // Kiểm tra định dạng file
  const ALLOWED_TYPES = [
    'application/pdf',
    'application/vnd.openxmlformats-officedocument.wordprocessingml.document',
    'text/plain',
    'text/markdown',
    'text/csv',
    'application/json',
    'image/jpeg',
    'image/png',
  ];

  for (const file of files) {
    if (!ALLOWED_TYPES.includes(file.type)) {
      setError(`File ${file.name} không được hỗ trợ`);
      return false;
    }
  }

  return true;
};
```

### 8.2. API Error Handling

```typescript
// API Client error interceptor
this.client.interceptors.response.use(
  (response) => response,
  (error) => {
    if (error.response) {
      // Server trả về lỗi
      const status = error.response.status;
      const message = error.response.data?.detail || 'Đã xảy ra lỗi';

      switch (status) {
        case 400:
          throw new Error(`Dữ liệu không hợp lệ: ${message}`);
        case 401:
          // Xóa token và chuyển đến trang login
          localStorage.removeItem('auth_token');
          window.location.href = '/login';
          throw new Error('Phiên đăng nhập hết hạn');
        case 403:
          throw new Error('Không có quyền truy cập');
        case 404:
          throw new Error('Không tìm thấy tài nguyên');
        case 429:
          throw new Error('Quá nhiều yêu cầu, vui lòng thử lại sau');
        case 500:
          throw new Error(`Lỗi server: ${message}`);
        default:
          throw new Error(message);
      }
    } else if (error.request) {
      // Request được gửi nhưng không nhận được phản hồi
      throw new Error('Không thể kết nối đến server');
    } else {
      // Lỗi khác
      throw new Error(error.message);
    }
  }
);
```

### 8.3. UI Error Display

```typescript
// Chat.tsx
const handleSendMessage = async (message: string, files: File[] = []) => {
  try {
    // ... xử lý gửi tin nhắn
  } catch (error) {
    const errorMessage = error instanceof Error
      ? error.message
      : 'Đã xảy ra lỗi không xác định';

    setError(errorMessage);

    // Tự động ẩn lỗi sau 5 giây
    setTimeout(() => {
      setError(null);
    }, 5000);
  } finally {
    setLoading(false);
  }
};

// Render error message
{error && (
  <div className="error-banner">
    <span className="error-icon">⚠️</span>
    <span className="error-message">{error}</span>
    <button onClick={() => setError(null)}>✕</button>
  </div>
)}
```

---

## 9. BẢO MẬT VÀ XÁC THỰC

### 9.1. JWT Authentication

**Token Storage:**
```typescript
// Login thành công
const loginResponse = await apiClient.login(username, password);
localStorage.setItem('auth_token', loginResponse.access_token);
localStorage.setItem('user', JSON.stringify(loginResponse.user));
```

**Token Injection:**
```typescript
// API Client interceptor
this.client.interceptors.request.use((config) => {
  const token = localStorage.getItem('auth_token');
  if (token) {
    config.headers.Authorization = `Bearer ${token}`;
  }
  return config;
});
```

**Token Format:**
```
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

### 9.2. CORS Configuration

**Allowed Origins:**
- `http://localhost:6009` (Frontend dev)
- `http://localhost:6010` (Alternative)
- `http://localhost:6008` (Backend)

**Allowed Methods:**
- GET, POST, PUT, DELETE, OPTIONS, PATCH

**Allowed Headers:**
- Authorization, Content-Type, Accept

### 9.3. Secure Data Handling

**Không lưu trữ:**
- Password (chỉ gửi khi login)
- Sensitive metadata

**Lưu trữ an toàn:**
- JWT token trong localStorage (auto-expiry sau 24h)
- User info cơ bản

---

## 10. PERFORMANCE OPTIMIZATION

### 10.1. State Management Optimization

```typescript
// Sử dụng Zustand với immer middleware
import { create } from 'zustand';
import { immer } from 'zustand/middleware/immer';

export const useChatStore = create(
  immer<ChatState>((set) => ({
    // ... state
    addMessage: (message) =>
      set((state) => {
        state.currentMessages.push(message);
      }),
  }))
);
```

### 10.2. Component Memoization

```typescript
// MessageBubble.tsx
import { memo } from 'react';

const MessageBubble = memo(({ message }: { message: ChatMessage }) => {
  return (
    <div className={`message-bubble ${message.role}`}>
      {message.content}
    </div>
  );
});

export default MessageBubble;
```

### 10.3. Lazy Loading

```typescript
// App.tsx
import { lazy, Suspense } from 'react';

const Chat = lazy(() => import('./pages/Chat'));
const Settings = lazy(() => import('./pages/Settings'));

function App() {
  return (
    <Suspense fallback={<LoadingSpinner />}>
      <Routes>
        <Route path="/chat" element={<Chat />} />
        <Route path="/settings" element={<Settings />} />
      </Routes>
    </Suspense>
  );
}
```

### 10.4. Debounce User Input

```typescript
// MessageInput.tsx
import { useDebouncedCallback } from 'use-debounce';

const handleTyping = useDebouncedCallback((text: string) => {
  // Xử lý auto-save draft
  saveDraft(text);
}, 1000);
```

---

## 11. KẾT LUẬN

### 11.1. Tóm tắt thiết kế

Module Chat phía client được thiết kế theo kiến trúc component-based với:
- **State Management**: Zustand cho quản lý state tập trung
- **API Layer**: Axios client với interceptors cho authentication
- **UI Components**: React components tái sử dụng
- **Type Safety**: TypeScript cho type checking
- **Error Handling**: Xử lý lỗi toàn diện ở mọi layer

### 11.2. Điểm mạnh

1. **Tách biệt rõ ràng**: UI, Logic, API, State
2. **Type-safe**: TypeScript đảm bảo an toàn kiểu dữ liệu
3. **Scalable**: Dễ dàng mở rộng thêm tính năng
4. **Reusable**: Components và hooks có thể tái sử dụng
5. **Maintainable**: Code dễ đọc, dễ bảo trì

### 11.3. API Endpoints liên quan

| Method | Endpoint | Mô tả |
|--------|----------|-------|
| POST | `/api/v1/generate/chat` | Gửi tin nhắn và nhận phản hồi |
| POST | `/api/v1/chat-history/sessions` | Tạo phiên chat mới |
| GET | `/api/v1/chat-history/sessions` | Lấy danh sách phiên chat |
| GET | `/api/v1/chat-history/sessions/:id` | Lấy chi tiết phiên chat |
| PATCH | `/api/v1/chat-history/sessions/:id` | Cập nhật phiên chat |
| DELETE | `/api/v1/chat-history/sessions/:id` | Xóa phiên chat |
| POST | `/api/v1/chat-history/sessions/:id/messages` | Thêm tin nhắn |
| GET | `/api/v1/chat-history/sessions/:id/messages` | Lấy danh sách tin nhắn |
| POST | `/api/v1/chat-history/sessions/:id/summarize` | Tạo tóm tắt phiên chat |
| POST | `/api/v1/vectordb/upload/file` | Upload file |

### 11.4. Data Flow Summary

```
User Input → UI Component → ChatPage Handler → API Client
    ↓                                              ↓
State Update ← Parse Response ← HTTP Response ← Backend API
    ↓
Component Re-render → Display to User
```

---

## PHU LUC

### A. Cấu trúc thư mục Frontend

```
frontend/
├── src/
│   ├── api/
│   │   └── client.ts              # API Client (Axios)
│   ├── components/
│   │   └── __ami__/
│   │       ├── ChatHeader.tsx     # Header component
│   │       ├── ChatSidebar.tsx    # Sidebar component
│   │       ├── MessageList.tsx    # Danh sách tin nhắn
│   │       ├── MessageBubble.tsx  # Tin nhắn đơn
│   │       ├── MessageInput.tsx   # Ô nhập tin nhắn
│   │       └── SettingsPanel.tsx  # Panel cấu hình
│   ├── pages/
│   │   └── Chat.tsx               # Trang chat chính
│   ├── store/
│   │   └── chatStore.ts           # Zustand store
│   ├── styles/
│   │   └── __ami__/
│   │       ├── Chat.css
│   │       ├── ChatHeader.css
│   │       └── ChatSidebar.css
│   └── main.tsx                   # Entry point
├── package.json
└── vite.config.ts
```

### B. Environment Variables

```env
# Frontend (.env)
VITE_API_URL=http://localhost:6008/api/v1
```

### C. Dependencies

```json
{
  "dependencies": {
    "react": "^18.2.0",
    "react-dom": "^18.2.0",
    "axios": "^1.6.0",
    "zustand": "^4.4.0",
    "lucide-react": "^0.294.0"
  },
  "devDependencies": {
    "@types/react": "^18.2.0",
    "@types/react-dom": "^18.2.0",
    "@vitejs/plugin-react": "^4.2.0",
    "typescript": "^5.3.0",
    "vite": "^5.0.0"
  }
}
```

---

**Người thiết kế**: AMI Development Team
**Ngày tạo**: 2025-01-25
**Phiên bản**: 1.0.0
**Trạng thái**: Approved
