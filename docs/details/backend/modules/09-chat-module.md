# Chat Module

**Version:** v2  
**Date:** 2026-02-25  
**Status:** 📝 To Be Implemented  
**Pattern:** Follow [04-projects-module.md](04-projects-module.md) + WebSocket + RAG pipeline

---

## 🎯 Overview

Chat là **interface layer** — giao diện hội thoại giữa User và Agent.

```
Chat = Conversation Container + Message Stream + Context Building + Response Delivery
```

Chat **không phải** Agent, **không phải** Workflow. Chat chỉ là kênh giao tiếp:

```
User → Chat → Agent
Agent → (process) → Chat → User
```

---

## 🧬 Phân loại Chat System

### 1. Simple Chat

- **Mô tả:** 1 user ↔ 1 agent, stateless (chỉ recent messages)
- **Đặc điểm:** Không có RAG, không có tool calling, chỉ text in/out
- **Use case:** Quick Q&A, brainstorming, casual conversation với agent
- **Context window:** Last N messages from Redis sliding window

### 2. RAG Chat

- **Mô tả:** Inject document context vào conversation
- **Đặc điểm:**
  - Vector search trước khi gửi LLM
  - Response có `sources` (trích dẫn tài liệu gốc)
  - Chỉ search trong docs thuộc org (tenant isolation)
- **Use case:** Hỏi đáp từ spec documents, wiki, project docs
- **Flow:** User message → VectorDB search → Top-K chunks → Augment prompt → LLM → Response + Sources

### 3. Tool-enabled Chat

- **Mô tả:** Agent có thể call API / trigger actions trong conversation
- **Đặc điểm:**
  - Agent quyết định khi nào cần gọi tool
  - Tool results inject vào conversation flow
  - User thấy cả text response lẫn tool actions
- **Use case:** "Tạo task cho tôi", "Cập nhật status issue #123", "Search wiki về authentication"
- **Tools available:** `create_task`, `update_issue_status`, `search_wiki`, `query_project_data`

### 4. Multi-agent Chat *(Advanced — Phase 2 Later)*

- **Mô tả:** 1 user + nhiều agents thảo luận trong cùng conversation
- **Đặc điểm:**
  - Có **Moderator Agent** điều phối turn-taking
  - Mỗi agent reply với perspective/expertise riêng
  - User có thể @mention agent cụ thể
- **Use case:** Design review (PM + Dev + QA agents cùng review), brainstorming session
- **Flow:** User message → Moderator routes → Agent A responds → Agent B responds → Moderator synthesizes

### Chat Type Summary

| Type | RAG? | Tools? | Multi-agent? | Complexity |
|------|------|--------|-------------|------------|
| Simple | ❌ | ❌ | ❌ | Low |
| RAG | ✅ | ❌ | ❌ | Medium |
| Tool-enabled | ✅/❌ | ✅ | ❌ | Medium-High |
| Multi-agent | ✅/❌ | ✅/❌ | ✅ | High |

---

## 🏗️ Architecture

```
┌──────────────────────────────────────────────────────────┐
│                 Frontend (Chat UI)                        │
│  ├─ Message list + input                                  │
│  ├─ Agent selector                                        │
│  ├─ Source citations (RAG)                                │
│  └─ Tool action indicators                                │
└────────────┬───────────────────────┬─────────────────────┘
             │ REST (CRUD)           │ WebSocket (real-time)
┌────────────▼───────────┐ ┌────────▼──────────────────────┐
│   ChatController       │ │   ChatGateway (Socket.IO)     │
│   - CRUD chats         │ │   - handleMessage (stream)    │
│   - list messages      │ │   - handleTyping              │
│   - send message (REST)│ │   - auth on connection        │
└────────────┬───────────┘ └────────┬──────────────────────┘
             └──────────┬───────────┘
                        │
              ┌─────────▼─────────┐
              │    ChatService    │
              │  - sendMessage()  │
              │  - sendStream()   │
              │  - getHistory()   │
              └─────────┬─────────┘
                        │
         ┌──────────────┼──────────────┐
         ▼              ▼              ▼
   ┌──────────┐  ┌───────────┐  ┌──────────┐
   │  Redis   │  │ AgentSvc  │  │ MySQL    │
   │ (buffer) │  │ (execute) │  │ (persist)│
   └──────────┘  └───────────┘  └──────────┘
```

---

## 📋 DTOs & Validation

### CreateChatDTO

```typescript
export const CreateChatSchema = z.object({
  agent_id: z.string().uuid(),
  title: z.string().max(200).optional(),         // Auto-generated from first message if empty
  chat_type: z.enum(['simple', 'rag', 'tool_enabled', 'multi_agent']).default('simple'),

  // Multi-agent config
  multi_agent_config: z.object({
    moderator_agent_id: z.string().uuid(),
    participant_agent_ids: z.array(z.string().uuid()).min(1).max(5),
  }).optional(),

  // Context
  project_id: z.string().uuid().optional(),       // Scope chat to project
})
```

### SendMessageDTO

```typescript
export const SendMessageSchema = z.object({
  content: z.string().min(1).max(10000),
  target_agent_id: z.string().uuid().optional(),   // For multi-agent: @mention specific agent
  attachments: z.array(z.object({
    type: z.enum(['document_id', 'file_url']),
    value: z.string(),
  })).optional(),
})
```

### ChatResponseDTO

```typescript
export interface ChatResponseDTO {
  id: string
  title: string
  chat_type: 'simple' | 'rag' | 'tool_enabled' | 'multi_agent'
  agent: { id: string; name: string; avatar_url?: string }
  project_id?: string

  // Multi-agent
  participants?: Array<{ id: string; name: string; avatar_url?: string }>

  message_count: number
  last_message_at: string | null
  created_at: string
  updated_at: string
}
```

### MessageResponseDTO

```typescript
export interface MessageResponseDTO {
  id: string
  chat_id: string
  role: 'user' | 'assistant' | 'system' | 'tool'
  content: string
  agent_id?: string              // Which agent responded (multi-agent)
  agent_name?: string

  // RAG sources
  sources?: Array<{
    document_id: string
    document_title: string
    chunk_preview: string        // 200 chars
    similarity_score: number
  }>

  // Tool calls
  tool_calls?: Array<{
    tool_name: string
    arguments: Record<string, any>
    result: any
    status: 'success' | 'failed'
  }>

  // Meta
  token_count?: number
  provider_model?: string
  created_at: string
}
```

---

## 🔌 API Routes

```
# ── Chat CRUD ──
GET     /api/chats                     → List user's chats (with filters: agent_id, chat_type)
POST    /api/chats                     → Create new chat
GET     /api/chats/:id                 → Get chat detail
PUT     /api/chats/:id                 → Update chat (title)
DELETE  /api/chats/:id                 → Delete chat + all messages

# ── Messages ──
GET     /api/chats/:id/messages        → List messages (paginated, newest first)
POST    /api/chats/:id/messages        → Send message (REST, non-streaming)

# ── WebSocket Events ──
# Client → Server
# - 'join_chat'     { chatId }
# - 'send_message'  { chatId, content, targetAgentId? }
# - 'typing'        { chatId }
#
# Server → Client
# - 'message_chunk' { chunk, agentId? }
# - 'message_complete' { messageId, sources?, toolCalls? }
# - 'agent_typing'  { chatId, agentId }
# - 'error'         { message }
```

---

## 🧩 Service Layer

### ChatService

```typescript
export class ChatService {
  constructor(
    private readonly chatRepo: ChatRepository,
    private readonly messageRepo: MessageRepository,
    private readonly redis: RedisService,
    private readonly agentService: AgentService
  ) {}

  /**
   * Send message with Redis Sliding Window Context
   * Flow:
   * 1. Get recent context from Redis (fast)
   * 2. If Redis empty → refill from MySQL (cache refill)
   * 3. Route to Agent based on chat_type
   * 4. Persist both user + assistant messages
   * 5. Update Redis sliding window
   */
  async sendMessage(organizationId: string, chatId: string, userId: string, content: string): Promise<MessagePair>

  /**
   * Send message with streaming (for WebSocket gateway)
   * Same as sendMessage but yields chunks via AsyncGenerator
   */
  async *sendMessageStream(organizationId: string, chatId: string, userId: string, content: string): AsyncGenerator<StreamChunk>

  /**
   * Multi-agent: Route message to all participant agents
   * 1. Moderator decides which agents should respond
   * 2. Execute each agent sequentially
   * 3. Return array of responses
   */
  async sendMultiAgentMessage(chatId: string, userId: string, content: string): Promise<MessageResponseDTO[]>
}
```

### Chat Context Management

```typescript
/**
 * Redis Sliding Window cho chat context
 *
 * Key: wh:{orgId}:chat:{chatId}:buffer
 * - rpush: thêm message mới
 * - ltrim: giữ last 10 messages
 * - lrange: đọc toàn bộ buffer
 * - TTL: 24h (auto-cleanup inactive chats)
 *
 * Khi Redis empty → refill từ MySQL (last 10 messages)
 */
```

---

## 🔒 Security Considerations

- ✅ **Tenant isolation in RAG** — Chỉ search documents thuộc org
- ✅ **Chat ownership** — User chỉ access chat của mình
- ✅ **WebSocket auth** — Verify JWT on connection + per-message
- ✅ **Rate limiting** — Prevent AI abuse (per user per minute)
- ✅ **Content filtering** — PII detection, harmful content blocking
- ✅ **Message size limit** — Max 10,000 chars per message
- ✅ **Multi-agent guard** — Max 5 agents per chat, moderator required

---

## 📚 Related Documents

- [08-agents-module.md](08-agents-module.md) — AI agent definitions & execution
- [10-workflows-module.md](10-workflows-module.md) — Workflow integration
- [06-documents-module.md](06-documents-module.md) — Vector DB / RAG source
- [04-projects-module.md](04-projects-module.md) — Base pattern

---

*Last Updated: 2026-02-25 — v2 (Added Chat classification, DTOs, WebSocket events, Multi-agent Chat)*
