# AI & Scale-up UI (Phase 2)

**Status:** 📝 Planned / Scale-up  
**Tech:** WebSockets, Streaming UI, Framer Motion, React Flow

---

## 🤖 AI Agent Management UI

### 1. Agent List Page (`/agents`)

- **Card Grid Layout** — Mỗi Agent hiển thị như card với avatar, name, role badge, model info
- **Status Toggle** — Active/Disabled trực tiếp từ list
- **Quick Stats** — Hiện total executions, tokens used, last used
- **Filters** — Theo role (PM, Dev, Reviewer, Analyst, Writer), status, project
- **Actions** — Create, Edit, Clone, Delete

### 2. Agent Detail / Edit Page (`/agents/:id/edit`)

**Main Content:**
- Name, Description
- System Prompt (textarea lớn, có syntax highlighting)
- Prompt preview/test panel

**Sidebar:**
- **Role Selector** — Dropdown: PM, Developer, Reviewer, Analyst, Writer, Custom
- **Model Configuration:**
  - Provider dropdown (OpenAI, Anthropic, Google, Ollama, vLLM)
  - Model input (gpt-4o, claude-3.5-sonnet, gemini-2.0-flash, llama3)
  - Custom Base URL (cho self-hosted)
  - AI Parameters: Temperature slider, Max tokens input, Top P slider
- **Knowledge Base (Spec Hub):**
  - Document picker (multi-select từ Documents list)
  - RAG config toggle: Enabled/Disabled
  - Top K slider, Similarity threshold slider
- **Tools:**
  - Checkbox list: create_task, update_issue_status, search_wiki, get_document_content, v.v.
- **Status:** Active / Disabled toggle

### 3. Agent Test Panel

- Input textarea để nhập test prompt
- "Execute" button → streaming response
- Hiển thị: response text, tokens used, latency
- Không lưu vào chat history

---

## 🔄 AI Workflow UI

### 1. Workflow List Page (`/workflows`)

- **Table/Card Layout** — Name, description, trigger type badge, step count, total runs
- **Status Badge** — Draft, Active, Archived
- **Quick Actions** — Run, Edit, Clone, View History

### 2. Workflow Builder Page (`/workflows/:id/edit`)

**Visual Pipeline Editor (React Flow):**

```
┌──────────┐    ┌──────────┐    ┌──────────┐
│  Step 1  │───→│  Step 2  │───→│  Step 3  │
│ Analyst  │    │   PM     │    │   Dev    │
│ GPT-4o   │    │ Claude   │    │ Gemini   │
└──────────┘    └──────────┘    └──────────┘
```

- **Drag & Drop Steps** — Thêm/sắp xếp lại các bước
- **Step Config Panel** (slide-out khi click vào step):
  - Agent selector (dropdown)
  - Model override (optional)
  - Prompt template editor (hỗ trợ `{{variables}}`)
  - Input mapping source
  - Conditions
  - Timeout & Retry settings
- **Trigger Config Panel:**
  - Manual / Scheduled (cron picker) / Event (dropdown: issue_created, document_updated)
- **Input Schema Builder:**
  - Drag & drop form fields (text, textarea, document picker, project picker)

### 3. Workflow Run Page (`/workflows/:id/run`)

- **Input Form** — Dynamic form từ `input_schema`
- **"Run Workflow" Button** → starts execution

### 4. Workflow Instance Detail (`/workflows/instances/:id`)

**Real-time Execution Tracker:**

```
✅ Step 1: Analyze Requirements       │ GPT-4o   │ 1523 tokens │ 8.4s
🔄 Step 2: Create User Stories        │ Claude   │ running...  │ --
⬜ Step 3: Generate Tasks             │ Gemini   │ pending     │ --
```

- **Per-step expandable panels:**
  - Input sent (collapsible)
  - Output received (full markdown render)
  - Model used, tokens, latency
  - Error details (if failed)
- **Actions:** Cancel (if running), Retry (if failed)
- **Aggregate Stats:** Total tokens, total duration, status

### 5. Workflow History Page (`/workflows/:id/instances`)

- **Table:** Date, status, triggered by, duration, tokens used
- **Compare Instances** — Side-by-side diff view giữa 2 lần chạy

---

## 💬 AI Chat Interface

### Streaming Responses
- Hiển thị text theo thời gian thực (kiểu ChatGPT)
- Markdown rendering: code highlighting, tables, LaTeX
- RAG citations: Hiển thị nguồn document mà AI tham khảo

### Agent-aware Chat
- Mỗi chat gắn với 1 Agent cụ thể
- Hiển thị Agent avatar + name trong header
- Tool calls hiển thị như action cards

---

## 📈 Optimization for Scale

- **Edge Runtime** — Cho API chat cần latency thấp
- **WebSocket** — Streaming responses, real-time workflow progress
- **React Flow** — Workflow visual editor (node-based)
- **Framer Motion** — Micro-interactions, step transitions

---

*Last Updated: 2026-02-25*
