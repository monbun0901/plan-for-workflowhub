# AI Gateway (Orchestration Layer)

**Status:** 🏗️ Phase 2 Design  
**Core Responsibility:** Một lớp trung gian điều phối cuộc gọi giữa Backend, Knowledge Base (RAG) và các mô hình ngôn ngữ lớn (LLM).

---

## 🏗️ The AI Pipeline

Khi một yêu cầu chat tới, AI Gateway thực hiện các bước sau:

1.  **Context Enrichment (Làm giàu ngữ cảnh):**
    *   Lấy cấu hình Agent từ MySQL (System Prompt, Tools).
    *   Lấy 10 tin nhắn gần nhất từ **Redis Buffer**.
    *   Gọi **RAG Service** để lấy tri thức nội bộ liên quan.
2.  **Prompt Baking (Nướng Prompt):** Trộn tất cả thông tin thành một Prompt hoàn chỉnh theo cấu trúc chuyên nghiệp.
3.  **LLM Execution:** Gửi Prompt tới OpenAI/Claude/Local LLM.
4.  **Result Processing:** 
    *   Xử lý phản hồi (Text hoặc JSON Tool Call).
    *   Lưu vết (Logging) và tính toán Token sử dụng.

---

## 🧩 Key Features

### 1. Prompt Constructor (Bản thiết kế Prompt)
Xây dựng prompt theo cấu trúc:
- **System:** `[Agent Core Instructions] + [Project Constraints]`
- **Knowledge:** `[Context from RAG Service]`
- **History:** `[Last 10 messages from Redis]`
- **User:** `[Current User Message]`

### 2. Multi-Model Support
Khả năng chuyển đổi linh hoạt giữa các model mà không làm thay đổi logic nghiệp vụ:
*   **High Performance:** GPT-4o, Claude 3.5 Sonnet.
*   **Cost-Effective:** GPT-4o-mini, Haiku.

### 3. Tool Calling (Hành động của AI)
Định nghĩa danh sách các hàm mà AI có thể gọi:
- `create_task(name, priority)`
- `update_issue_status(id, status)`
- `search_wiki(query)`

### 4. Logging & Monitoring
*   Lưu toàn bộ Input/Output (trừ dữ liệu nhạy cảm) để Audit.
*   Theo dõi `latency` (độ trễ) và `token usage` của từng Organization.

---

## 🔒 Security

*   **Prompt Injection Protection:** Lọc và kiểm tra các ký tự lạ hoặc các yêu cầu cố tình phá vỡ System Prompt.
*   **API Key Management:** Toàn bộ API keys của OpenAI/Claude được lưu an toàn tại Environment Variables phía Backend, không bao giờ lộ ra Frontend.

---

## 🔌 OpenClaw Integration (Agent Engine)

WorkflowHub sử dụng **OpenClaw** làm bộ máy thực thi Agent mạnh mẽ cho các tác vụ phức tạp (Action-oriented tasks).

### 1. Connection Architecture
Chúng ta kết nối WorkflowHub Backend với OpenClaw Server thông qua **WebSocket Gateway**.

```
[ WorkflowHub UI ] <───> [ WH Backend ] <───(WS)───> [ OpenClaw Engine ]
                                │                            │
                        [ RAG Context ]              [ Action Execution ]
```

### 2. Integration Logic (The Bridge)
Khi nhận yêu cầu, WH Backend đóng vai trò là "Provider" cho OpenClaw:
*   **Context Injection:** Gửi kèm tri thức từ RAG Service của WorkflowHub vào OpenClaw Prompt.
*   **Tool Mapping:** Map các Tools của WorkflowHub (create_task, search_wiki) vào danh sách "Capabilities" của OpenClaw.
*   **Event Routing:** Kết nối sự kiện từ Discord (nếu có) về hệ thống thông báo của WorkflowHub.

### 3. Benefits for WorkflowHub
*   **Multi-Platform:** Đồng bộ hóa trải nghiệm chat giữa Web App và Discord Bot.
*   **Advanced Reasoning:** Tận dụng khả năng tự suy nghĩ các bước thực hiện (Task decomposition) có sẵn của OpenClaw.
*   **Ecosystem Access:** Sử dụng ngay các tích hợp sẵn có của OpenClaw (GitHub, Gmail, Calendar).

### 4. Dynamic Agent Injection (No-Config Implementation)
Để hỗ trợ Multi-tenant, chúng ta không dùng file cấu hình `.openclaw` tĩnh. Thay vào đó, Backend sẽ "bơm" cấu hình trực tiếp qua WebSocket:

*   **Payload structure:** Mỗi request gửi sang OpenClaw sẽ bao gồm một `agent_manifest`:
    ```json
    {
      "session_id": "...",
      "agent_config": {
        "system_prompt": "Defined in WH Database",
        "model": "gpt-4o",
        "tools": ["create_task", "search_wiki"],
        "rag_context": "Relevant text chunks from WH RAG Service"
      },
      "message": "User's current query"
    }
    ```
*   **Stateless Engine:** OpenClaw Engine đóng vai trò là một "bộ xử lý thuần túy", nhận cấu hình từ WorkflowHub, thực thi và trả kết quả, không giữ cấu hình cố định trong file cục bộ.

---

## 🚀 Internal Workflow Example (with OpenClaw)

```typescript
async handleChatMessage(chatId, message) {
  // 1. Get Agent & Tools
  const agent = await agentRepo.getForChat(chatId);
  
  // 2. Get Context (RAG + Redis)
  const [knowledge, history] = await Promise.all([
    ragService.query(message, orgId),
    redis.getHistory(chatId)
  ]);
  
  // 3. Call LLM via Gateway
  const response = await aiGateway.complete({
    system: agent.system_prompt,
    context: knowledge,
    history: history,
    user: message,
    tools: agent.tools
  });
  
  return response;
}
```

---

*Last Updated: 2026-02-11*
