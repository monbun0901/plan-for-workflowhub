# AI Agents Module

**Version:** v3  
**Date:** 2026-02-25  
**Status:** 📝 To Be Implemented  
**Pattern:** Follow [04-projects-module.md](04-projects-module.md) + Domain-specific logic

---

## 🎯 Overview

Mỗi **AI Agent** đại diện cho một **thực thể có vai trò + mục tiêu + luật + khả năng hành động** trong team.

```
Agent = Identity (Role + Prompt) + Knowledge (RAG) + Tools + Model Config + Rules/Guardrails
```

### Thành phần của Agent

| Thành phần | Mô tả |
|---|---|
| **Identity (System Prompt)** | Vai trò, phong cách trả lời, constraints — không expose cho client |
| **Rules / Guardrails** | Giới hạn hành vi: không tạo data ngoài scope, rate limit, token budget |
| **Knowledge (Spec Hub)** | Bộ tài liệu nội bộ (Documents) Agent được phép truy cập qua RAG |
| **Tools (Task Logic)** | Danh sách hàm Agent có thể gọi: `create_task`, `search_wiki`, `update_issue_status` |
| **Model Config** | LLM provider + model + parameters (temperature, max_tokens, top_p) |

---

## 🧬 Phân loại Agent

### 1. Conversational Agent

- **Mục đích:** Trả lời, tư vấn, giải đáp thắc mắc
- **Đặc điểm:** Dùng nhiều RAG, phản hồi dạng text, không chủ động trigger workflow
- **Ví dụ trong WorkflowHub:**
  - **Support Bot** — Trả lời câu hỏi dự án từ docs nội bộ
  - **Onboarding Assistant** — Hướng dẫn thành viên mới
- **Cấu hình tiêu biểu:** RAG enabled, tools minimal, temperature 0.5–0.7

### 2. Task Agent

- **Mục đích:** Thực hiện nhiệm vụ cụ thể, trả structured output
- **Đặc điểm:** Có output schema/validator, kết quả có thể persist vào DB
- **Ví dụ trong WorkflowHub:**
  - **Analyst Agent** — Phân tích spec → trả JSON requirements list
  - **Writer Agent** — Soạn báo cáo từ project data → Markdown format
  - **Reviewer Agent** — Review document → trả structured feedback
- **Cấu hình tiêu biểu:** `output_format: 'json' | 'markdown'`, tools enabled, temperature 0.3–0.5

### 3. Orchestrator Agent

- **Mục đích:** Điều phối agent khác — "AI Manager"
- **Đặc điểm:**
  - Phân tích yêu cầu phức tạp → quyết định gọi agent nào
  - Có thể trigger workflow
  - Tổng hợp kết quả từ nhiều agent
- **Ví dụ trong WorkflowHub:**
  - **PM Agent (Orchestrator)** — Nhận feature request → gọi Analyst → gọi Dev → tạo issues
  - **Review Coordinator** — Routing review request tới reviewer phù hợp
- **Cấu hình tiêu biểu:** tools = `['call_agent', 'trigger_workflow', 'route_request']`, temperature 0.2–0.4

### 4. Autonomous Agent *(Advanced — Phase 2 Later)*

- **Mục đích:** Hoạt động tự chủ với long-term goal
- **Đặc điểm:**
  - Có memory dài hạn (persistent across sessions)
  - Có reasoning loop: **Plan → Act → Observe → Repeat**
  - Tự quyết định bước tiếp theo
- **Ví dụ trong WorkflowHub:**
  - **Project Monitor** — Tự scan project health → cảnh báo risks → đề xuất actions
  - **Sprint Planner** — Tự phân tích backlog → suggest sprint plan
- **Cấu hình tiêu biểu:** `memory_enabled: true`, `goal: string`, `max_iterations: number`

### Agent Classification Summary

| Type | Trigger | Output | Gọi Agent khác? | Memory dài hạn? |
|------|---------|--------|-----------------|-----------------|
| Conversational | User message | Text (freeform) | ❌ | ❌ |
| Task | User/Workflow | Structured (JSON/MD) | ❌ | ❌ |
| Orchestrator | User/Event | Delegated results | ✅ | ❌ |
| Autonomous | Goal-driven | Actions + Reports | ✅ | ✅ |

---

## 🏗️ Architecture

```
┌─────────────────────────────────────────────────────────┐
│                     Frontend (Chat UI)                   │
└──────────────────────┬──────────────────────────────────┘
                       │ REST / WebSocket
┌──────────────────────▼──────────────────────────────────┐
│                  AgentController                         │
│  CRUD agents + /agents/:id/execute (chat endpoint)       │
└──────────────────────┬──────────────────────────────────┘
                       │
┌──────────────────────▼──────────────────────────────────┐
│                  AgentService                            │
│  - createAgent, updateAgent, deleteAgent                 │
│  - getAgentManifest (load prompt + docs + tools)         │
│  - validateAgentConfig (check model, provider, prompt)   │
│  - executeAgent → AIOrchestrator                         │
└──────────────────────┬──────────────────────────────────┘
                       │
          ┌────────────┼────────────┐
          ▼            ▼            ▼
    ┌──────────┐ ┌──────────┐ ┌──────────┐
    │  agents  │ │  agent_  │ │  agent_  │
    │  table   │ │documents │ │  tools   │
    └──────────┘ └──────────┘ └──────────┘
```

### Quan hệ Agent với các module khác

```
User → Chat → Agent (Conversational / Task)
Agent (Orchestrator) → (Tool) → Agent khác
Agent (Orchestrator) → (Tool) → Workflow
Workflow → Step → Agent (Task)
Agent (Autonomous) → Plan → Agent/Workflow → Observe → Loop
```

---

## 📋 DTOs & Validation

### CreateAgentDTO

```typescript
// apps/api/src/modules/agents/dtos/create-agent.dto.ts

export const CreateAgentSchema = z.object({
  name: z.string().min(1).max(100),
  description: z.string().max(500).optional(),

  // ── Role & Identity ──
  role: z.enum(['pm', 'developer', 'reviewer', 'analyst', 'writer', 'custom']),
  avatar_url: z.string().url().optional(),

  // ── Agent Type ──
  agent_type: z.enum(['conversational', 'task', 'orchestrator', 'autonomous']).default('conversational'),

  // ── Prompt Engineering ──
  system_prompt: z.string().min(10).max(10000),

  // ── Guardrails ──
  guardrails: z.object({
    max_tokens_per_request: z.number().min(100).max(32000).default(4096),
    max_requests_per_minute: z.number().min(1).max(100).default(20),
    allowed_topics: z.array(z.string()).optional(),      // Restrict conversation scope
    blocked_actions: z.array(z.string()).optional(),      // Actions agent cannot perform
  }).optional(),

  // ── Model Configuration ──
  provider: z.enum(['openai', 'anthropic', 'google', 'ollama', 'vllm']),
  model: z.string().min(1).max(100),     // 'gpt-4o', 'claude-3.5-sonnet', 'gemini-2.0-flash', 'llama3:8b'
  base_url: z.string().url().optional(),  // Custom endpoint (Ollama, vLLM)

  // ── AI Parameters ──
  ai_settings: z.object({
    temperature: z.number().min(0).max(2).default(0.7),
    max_tokens: z.number().min(100).max(32000).default(4096),
    top_p: z.number().min(0).max(1).default(1),
  }).optional(),

  // ── Knowledge Base (Spec Hub) ──
  document_ids: z.array(z.string().uuid()).optional(),  // IDs tài liệu gắn với Agent

  // ── RAG Configuration ──
  rag_config: z.object({
    enabled: z.boolean().default(true),
    top_k: z.number().min(1).max(20).default(5),
    similarity_threshold: z.number().min(0).max(1).default(0.7),
  }).optional(),

  // ── Tools (Task Logic) ──
  enabled_tools: z.array(z.string()).optional(),  // ['create_task', 'search_wiki', 'update_issue_status']

  // ── Orchestrator-specific ──
  orchestrator_config: z.object({
    delegatable_agent_ids: z.array(z.string().uuid()),   // Agents this orchestrator can call
    workflow_ids: z.array(z.string().uuid()).optional(),  // Workflows this orchestrator can trigger
    routing_strategy: z.enum(['auto', 'round_robin', 'capability_match']).default('auto'),
  }).optional(),

  // ── Autonomous-specific ──
  autonomous_config: z.object({
    goal: z.string().max(1000),
    max_iterations: z.number().min(1).max(50).default(10),
    memory_enabled: z.boolean().default(true),
    observation_interval_ms: z.number().min(60000).default(300000), // 5 min default
  }).optional(),

  // ── Output Configuration (Task Agent) ──
  output_config: z.object({
    format: z.enum(['text', 'json', 'markdown']).default('text'),
    schema: z.any().optional(),              // JSON Schema for structured output validation
    post_action: z.enum(['none', 'create_issue', 'create_task', 'update_document']).default('none'),
  }).optional(),

  // ── Scope ──
  project_id: z.string().uuid().optional(),  // Gắn Agent vào project cụ thể
})
```

### UpdateAgentDTO

```typescript
export const UpdateAgentSchema = CreateAgentSchema.partial().omit({ project_id: true })
```

### AgentResponseDTO

```typescript
export interface AgentResponseDTO {
  id: string
  name: string
  description?: string
  role: string
  agent_type: 'conversational' | 'task' | 'orchestrator' | 'autonomous'
  avatar_url?: string

  // Model info
  provider: string
  model: string

  // Guardrails
  guardrails?: {
    max_tokens_per_request: number
    max_requests_per_minute: number
    allowed_topics?: string[]
    blocked_actions?: string[]
  }

  // Knowledge
  knowledge_base: Array<{ id: string; title: string }>
  enabled_tools: string[]

  // Type-specific config
  orchestrator_config?: {
    delegatable_agent_ids: string[]
    workflow_ids?: string[]
    routing_strategy: string
  }
  autonomous_config?: {
    goal: string
    max_iterations: number
    memory_enabled: boolean
  }
  output_config?: {
    format: string
    schema?: any
    post_action: string
  }

  // Meta
  status: 'active' | 'disabled'
  usage_stats?: {
    total_executions: number
    total_tokens_used: number
    last_used_at: string | null
  }

  created_at: string
  updated_at: string
}
```

---

## 🔌 API Routes

```
GET     /api/agents                    → List agents (with filters: role, agent_type, status)
POST    /api/agents                    → Create agent
GET     /api/agents/:id                → Get agent detail
PUT     /api/agents/:id                → Update agent
DELETE  /api/agents/:id                → Delete agent
PATCH   /api/agents/:id/status         → Toggle active/disabled

# Knowledge Base (Spec Hub)
GET     /api/agents/:id/documents      → List agent's knowledge documents
POST    /api/agents/:id/documents      → Attach documents to agent
DELETE  /api/agents/:id/documents/:docId → Detach document from agent

# Execution
POST    /api/agents/:id/execute        → Execute agent (single prompt → response)
POST    /api/agents/:id/test           → Test agent with sample prompt (no history saved)

# Orchestrator-specific
POST    /api/agents/:id/delegate       → Orchestrator delegates task to sub-agent
GET     /api/agents/:id/delegates      → List delegatable agents

# Clone
POST    /api/agents/:id/clone          → Clone agent config
```

---

## 🧩 Service Layer

### AgentService (Key Methods)

```typescript
export class AgentService {
  /**
   * Tạo Agent mới + gắn knowledge documents + đăng ký tools
   * Validate agent_type-specific config (orchestrator_config, autonomous_config, output_config)
   */
  async createAgent(data: CreateAgentDTO): Promise<Agent>

  /**
   * Load Agent config đầy đủ cho execution:
   * - System prompt + guardrails
   * - Knowledge documents (for RAG query)
   * - Enabled tools
   * - Model config
   * - Type-specific config (orchestrator delegates, output schema, etc.)
   */
  async getAgentManifest(agentId: string): Promise<AgentManifest>

  /**
   * Execute: Gửi prompt tới AI Gateway với đầy đủ context
   * Flow phụ thuộc agent_type:
   * - Conversational: Load manifest → RAG query → Build prompt → Call LLM → Return text
   * - Task: Load manifest → RAG query → Build prompt → Call LLM → Validate output → Post-action
   * - Orchestrator: Load manifest → Analyze request → Route to sub-agent(s) → Aggregate results
   * - Autonomous: Load manifest → Load memory → Plan → Act → Observe → Loop until goal met
   */
  async executeAgent(agentId: string, userMessage: string, chatId?: string): Promise<AgentResponse>

  /**
   * Orchestrator: Delegate task to a sub-agent
   */
  async delegateToAgent(orchestratorId: string, targetAgentId: string, task: string): Promise<AgentResponse>

  /**
   * Validate agent config trước khi save:
   * - Provider, model hợp lệ?
   * - System prompt trong giới hạn?
   * - Orchestrator config: delegatable agents tồn tại và active?
   * - Autonomous config: goal không rỗng?
   * - Output config: JSON schema hợp lệ?
   */
  async validateConfig(data: CreateAgentDTO): Promise<ValidationResult>

  /**
   * Clone Agent (tạo bản sao từ Agent có sẵn)
   */
  async cloneAgent(agentId: string, newName: string): Promise<Agent>
}
```

### AgentManifest (Internal DTO for execution)

```typescript
interface AgentManifest {
  agent_id: string
  agent_type: 'conversational' | 'task' | 'orchestrator' | 'autonomous'
  system_prompt: string
  provider: string
  model: string
  base_url?: string
  ai_settings: { temperature: number; max_tokens: number; top_p: number }
  guardrails: { max_tokens_per_request: number; max_requests_per_minute: number }
  rag_config: { enabled: boolean; top_k: number; similarity_threshold: number }
  enabled_tools: ToolDefinition[]
  knowledge_document_ids: string[]

  // Type-specific
  orchestrator_config?: { delegatable_agent_ids: string[]; workflow_ids: string[]; routing_strategy: string }
  autonomous_config?: { goal: string; max_iterations: number; memory_enabled: boolean }
  output_config?: { format: string; schema?: any; post_action: string }
}
```

---

## 🧠 Agent Execution Flow

### Conversational Agent Flow

```
User Message → AgentController.execute()
  │
  ├─ 1. Load AgentManifest (prompt, model, tools, rag config)
  │
  ├─ 2. Check Guardrails (rate limit, token budget)
  │
  ├─ 3. RAG Query (if enabled)
  │     └─ VectorDB search with user message → relevant chunks
  │
  ├─ 4. Build Full Prompt
  │     ├─ System: agent.system_prompt + guardrails instructions
  │     ├─ Knowledge: RAG chunks
  │     ├─ History: last N messages (from Redis)
  │     └─ User: current message
  │
  ├─ 5. Call AI Gateway → Route to correct provider/model
  │
  ├─ 6. Process Response → Text → return directly
  │
  └─ 7. Save to History + Audit Log
```

### Task Agent Flow

```
User/Workflow → AgentController.execute()
  │
  ├─ 1–5. (Same as Conversational)
  │
  ├─ 6. Process Response
  │     ├─ Parse output theo output_config.format (text/json/markdown)
  │     ├─ Validate output against output_config.schema (if JSON)
  │     └─ Retry once if validation fails
  │
  ├─ 7. Post-Action (if configured)
  │     ├─ create_issue → IssueService.create()
  │     ├─ create_task → TaskService.create()
  │     └─ update_document → DocumentService.update()
  │
  └─ 8. Return structured result + post-action status
```

### Orchestrator Agent Flow

```
User/Event → AgentController.execute()
  │
  ├─ 1. Load Orchestrator Manifest + delegatable agents list
  │
  ├─ 2. Analyze Request
  │     └─ LLM call: "Given this request, which agent(s) should handle it?"
  │
  ├─ 3. Route & Delegate
  │     ├─ routing_strategy = 'auto' → LLM picks agent
  │     ├─ routing_strategy = 'capability_match' → match by agent.role + tools
  │     └─ routing_strategy = 'round_robin' → distribute evenly
  │
  ├─ 4. Execute Sub-Agent(s) → AgentService.executeAgent()
  │
  ├─ 5. Aggregate Results
  │     └─ LLM call: "Synthesize results from agents into final answer"
  │
  └─ 6. Return aggregated response
```

### Autonomous Agent Flow *(Phase 2 Later)*

```
Goal → AutonomousRunner.start()
  │
  ├─ 1. Load Agent Manifest + Memory (persistent state)
  │
  ├─ 2. Plan
  │     └─ LLM: "Given goal + memory, what's the next action?"
  │
  ├─ 3. Act
  │     ├─ Execute tool / Call sub-agent / Trigger workflow
  │     └─ Record action in execution log
  │
  ├─ 4. Observe
  │     └─ LLM: "Did the action succeed? What did we learn? Is goal met?"
  │
  ├─ 5. Update Memory
  │     └─ Persist observations + learnings
  │
  ├─ 6. Loop Check
  │     ├─ Goal met? → Return final report
  │     ├─ Max iterations reached? → Return partial report
  │     └─ Continue? → Go to Step 2
  │
  └─ 7. Return execution summary
```

---

## 🛡️ Security Considerations

- ✅ **System prompt isolation** — Prompt không bao giờ expose cho client
- ✅ **Guardrails enforcement** — Rate limit + token budget per agent per user
- ✅ **Tool permission check** — Trước khi thực thi tool, verify user permission
- ✅ **Prompt injection protection** — Sanitize user input trước khi gửi LLM
- ✅ **API key management** — Keys lưu ở environment, không lưu DB
- ✅ **Orchestrator scope** — Orchestrator chỉ gọi được agents trong `delegatable_agent_ids`
- ✅ **Autonomous safety** — Max iterations + manual kill switch + approval gates

---

## 📚 Related Documents

- [10-workflows-module.md](10-workflows-module.md) — Agent cộng tác qua Workflow
- [09-chat-module.md](09-chat-module.md) — Chat UI cho Agent execution
- [../../services/02-ai-gateway.md](../../services/02-ai-gateway.md) — LLM orchestration layer
- [../../services/03-ai-tools.md](../../services/03-ai-tools.md) — Tool definitions
- [../../database/tables/agents.md](../../database/tables/agents.md) — DB schema

---

*Last Updated: 2026-02-25 — v3 (Added Agent classification, Guardrails, Orchestrator & Autonomous flows)*
