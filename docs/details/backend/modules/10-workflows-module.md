# AI Workflows Module

**Version:** v3  
**Date:** 2026-02-25  
**Status:** 📝 To Be Implemented  
**Pattern:** State Machine + Pipeline Orchestration

---

## 🎯 Overview

**AI Workflow** là **luồng điều khiển có cấu trúc (deterministic)** — định nghĩa cách các Agent cộng tác để hoàn thành quy trình nhiều bước.

```
Workflow = Graph of Nodes (Steps) + Trigger + Input/Output Schema + Execution Engine
```

### Workflow ≠ Agent

| Aspect | Agent | Workflow |
|--------|-------|----------|
| Logic | Linh hoạt (AI quyết định) | Cố định (graph xác định trước) |
| Thay đổi | Prompt engineering | Edit graph/steps |
| Traceability | Conversation history | Step-by-step execution log |
| Reusability | Clone agent | Run template nhiều lần |

### Quan hệ Agent ↔ Workflow

```
Agent có thể → được gọi trong workflow (as step executor)
Agent có thể → trigger workflow (via tool call)
Workflow      → gọi Agent tại mỗi step
Orchestrator Agent → trigger workflow thay vì tự xử lý
```

---

## 🧬 Phân loại Workflow

### 1. Linear Workflow

- **Mô tả:** A → B → C — chuỗi bước tuần tự
- **Đặc điểm:** Output bước trước = Input bước sau
- **Ví dụ:**
  - Gửi transcript → Summarize → Extract actions → Create tasks
  - Upload spec → Analyze → Breakdown user stories

```
Step 1 ──→ Step 2 ──→ Step 3 ──→ Output
```

### 2. Conditional Workflow

- **Mô tả:** Có nhánh if/else — quyết định bước tiếp theo dựa trên kết quả trước
- **Đặc điểm:**
  - `conditions` trên mỗi step evaluate trước khi chạy
  - Có thể skip step hoặc chọn branch
  - Support operators: `equals`, `not_equals`, `contains`, `exists`, `gt`, `lt`
- **Ví dụ:**
  - Nếu sentiment < 0 → escalate to human
  - Nếu complexity = 'high' → thêm step review
  - Nếu issue type = 'bug' → route to Dev Agent, else → PM Agent

```
Step 1 ──→ Condition? ──→ [Yes] Step 2A ──→ Step 3
                    └──→ [No]  Step 2B ──→ Step 3
```

### 3. Event-driven Workflow

- **Mô tả:** Tự động trigger từ event, không cần user bấm "Run"
- **Đặc điểm:**
  - **Webhook:** Nhận event từ external service
  - **Cron/Schedule:** Chạy định kỳ (daily summary, weekly report)
  - **Database change:** Issue created, document updated, status changed
  - **Agent output:** Agent hoàn thành task → trigger workflow tiếp
- **Ví dụ:**
  - Issue created → AI auto-categorize → assign team
  - Every Monday 9AM → scan open issues → generate priority report
  - Document updated → re-analyze → update related issues

```
Event (webhook/cron/DB change/agent output)
  └──→ Trigger ──→ Step 1 ──→ Step 2 ──→ ...
```

### 4. Human-in-the-loop Workflow

- **Mô tả:** Workflow dừng lại chờ user review/approve trước khi tiếp tục
- **Đặc điểm:**
  - Có node `wait_for_approval` / `wait_for_input`
  - Instance status = `awaiting_approval` khi đến node này
  - User approve/reject via API → workflow resume hoặc abort
  - Timeout configurable (auto-approve hoặc auto-cancel)
- **Ví dụ:**
  - AI draft email → user review → send
  - Auto-generated issues → PM approve → create in project
  - Estimated budget > $1000 → require manager approval

```
Step 1 ──→ Step 2 ──→ ⏸️ Wait for Approval ──→ Step 3 ──→ Output
                              │
                         User approves/rejects
```

### Workflow Type Summary

| Type | Logic cố định? | Trigger | Cần user giữa chừng? | Complexity |
|------|----------------|---------|----------------------|------------|
| Linear | ✅ Sequential | Manual | ❌ | Low |
| Conditional | ✅ Branching | Manual/Event | ❌ | Medium |
| Event-driven | ✅ Sequential/Branch | Webhook/Cron/DB | ❌ | Medium-High |
| Human-in-the-loop | ✅ + Pause node | Manual/Event | ✅ | High |

---

## 🏗️ Core Concepts

### 1. Workflow Template (Bản thiết kế)

Template là **định nghĩa tĩnh** của workflow, có thể tái sử dụng nhiều lần:

```
WorkflowTemplate = {
  name, description, category,
  workflow_type,     // 'linear' | 'conditional' | 'event_driven' | 'human_in_the_loop'
  trigger_config,    // Khi nào chạy?
  steps_config,      // Chạy những bước gì? Với Agent nào? Model nào?
  input_schema,      // Workflow cần input gì?
  output_schema,     // Workflow sẽ trả ra gì?
}
```

### 2. Workflow Instance (Lần chạy cụ thể)

Instance là **một lần thực thi** của template, lưu lại toàn bộ kết quả:

```
WorkflowInstance = {
  template_id,
  status: 'pending' | 'running' | 'awaiting_approval' | 'completed' | 'failed' | 'partial_success' | 'cancelled',
  input_data,            // Dữ liệu đầu vào
  execution_log,         // Log chi tiết từng bước
  step_results,          // Kết quả từng bước (input → output → tokens → latency)
  approval_requests,     // Human-in-the-loop: pending approvals
  total_tokens_used,
  total_duration_ms,
}
```

### 3. Workflow Step (Bước thực thi)

Mỗi step gán **một Agent cụ thể** với **một model cụ thể**:

```
WorkflowStep = {
  step_order: 1,
  name: "Phân tích yêu cầu",
  step_type: 'agent_call' | 'condition' | 'approval_gate' | 'delay' | 'webhook',
  agent_id: "analyst-agent-uuid",    // Required for 'agent_call'

  // Model override (ghi đè model mặc định của Agent)
  model_override: {
    provider: "openai",
    model: "gpt-4o"
  },

  // Prompt template cho step này (có thể dùng {{variables}})
  prompt_template: "Phân tích tài liệu sau:\n\n{{input}}\n\nTrả về danh sách yêu cầu...",

  // Input mapping: lấy dữ liệu từ đâu?
  input_mapping: {
    type: "previous_step" | "workflow_input" | "static" | "combined",
    source: "step_1.output" | "input.document_content"
  },

  // Conditions: bước này có chạy không? (Conditional Workflow)
  conditions: [
    { field: "step_1.output.has_requirements", operator: "equals", value: true }
  ],

  // Branching: nếu condition true → go to step X, else → step Y
  branch_config: {
    on_true: "step_3",       // Jump to step 3
    on_false: "step_4",      // Jump to step 4
  },

  // Human-in-the-loop: approval gate config
  approval_config: {
    approver_roles: ['admin', 'pm'],
    timeout_ms: 86400000,             // 24h
    timeout_action: 'auto_approve' | 'auto_cancel',
    message_template: "Please review AI output before proceeding",
  },

  // Execution settings
  timeout_ms: 60000,
  retry: { max_attempts: 2, delay_ms: 5000 }
}
```

---

## 📋 DTOs & Validation

### CreateWorkflowTemplateDTO

```typescript
export const CreateWorkflowTemplateSchema = z.object({
  name: z.string().min(1).max(100),
  description: z.string().max(1000).optional(),
  category: z.array(z.string()).optional(),

  // ── Workflow Type ──
  workflow_type: z.enum(['linear', 'conditional', 'event_driven', 'human_in_the_loop']).default('linear'),

  // ── Trigger ──
  trigger_config: z.object({
    type: z.enum(['manual', 'scheduled', 'event', 'webhook']),
    event: z.string().optional(),          // 'issue_created', 'document_updated'
    schedule: z.string().optional(),       // Cron: '0 9 * * 1' (Mon 9AM)
    webhook_config: z.object({
      url: z.string().url().optional(),
      secret: z.string().optional(),
    }).optional(),
  }),

  // ── Input Schema ──
  input_schema: z.object({
    fields: z.array(z.object({
      key: z.string(),
      type: z.enum(['text', 'textarea', 'document_id', 'project_id', 'file', 'select']),
      label: z.string(),
      required: z.boolean().default(true),
      options: z.array(z.string()).optional(),  // For 'select' type
    })),
  }).optional(),

  // ── Steps ──
  steps_config: z.array(z.object({
    step_order: z.number().min(1),
    name: z.string().min(1).max(100),
    step_type: z.enum(['agent_call', 'condition', 'approval_gate', 'delay', 'webhook']).default('agent_call'),
    agent_id: z.string().uuid().optional(),    // Required for 'agent_call'

    // Model override
    model_override: z.object({
      provider: z.enum(['openai', 'anthropic', 'google', 'ollama', 'vllm']),
      model: z.string(),
    }).optional(),

    // Prompt
    prompt_template: z.string().min(1).max(10000).optional(),

    // Input mapping
    input_mapping: z.object({
      type: z.enum(['previous_step', 'workflow_input', 'static', 'combined']),
      source: z.string().optional(),
      template: z.string().optional(),    // Handlebars template for 'combined'
    }).optional(),

    // Conditions
    conditions: z.array(z.object({
      field: z.string(),
      operator: z.enum(['equals', 'not_equals', 'contains', 'exists', 'gt', 'lt']),
      value: z.any(),
    })).optional(),

    // Branching
    branch_config: z.object({
      on_true: z.string().optional(),
      on_false: z.string().optional(),
    }).optional(),

    // Approval gate
    approval_config: z.object({
      approver_roles: z.array(z.string()),
      timeout_ms: z.number().min(60000).max(604800000).default(86400000),  // 1min – 7days
      timeout_action: z.enum(['auto_approve', 'auto_cancel']).default('auto_cancel'),
      message_template: z.string().max(1000).optional(),
    }).optional(),

    // Execution settings
    timeout_ms: z.number().min(5000).max(300000).default(60000),
    retry: z.object({
      max_attempts: z.number().min(0).max(5).default(1),
      delay_ms: z.number().min(1000).max(30000).default(5000),
    }).optional(),
  })).min(1).max(20),
})
```

### RunWorkflowDTO

```typescript
export const RunWorkflowSchema = z.object({
  template_id: z.string().uuid(),
  input_data: z.record(z.any()),           // Dynamic input based on input_schema
  project_id: z.string().uuid().optional(), // Context project
})
```

### ApprovalDecisionDTO

```typescript
export const ApprovalDecisionSchema = z.object({
  decision: z.enum(['approve', 'reject']),
  comment: z.string().max(1000).optional(),
  modified_data: z.any().optional(),        // User can modify AI output before approving
})
```

### WorkflowInstanceResponseDTO

```typescript
export interface WorkflowInstanceResponseDTO {
  id: string
  template: { id: string; name: string; workflow_type: string }
  status: 'pending' | 'running' | 'awaiting_approval' | 'completed' | 'failed' | 'partial_success' | 'cancelled'

  // Input/Output
  input_data: Record<string, any>

  // Step Results (truy vết từng bước)
  steps: Array<{
    step_order: number
    name: string
    step_type: string
    agent?: { id: string; name: string }
    model_used?: string
    status: 'pending' | 'running' | 'completed' | 'failed' | 'skipped' | 'awaiting_approval'
    input_preview: string        // 200 chars
    output: string | null
    tokens_used: number
    duration_ms: number
    error?: string

    // Approval info (for approval_gate steps)
    approval?: {
      status: 'pending' | 'approved' | 'rejected'
      decided_by?: { id: string; display_name: string }
      decided_at?: string
      comment?: string
    }

    started_at: string | null
    completed_at: string | null
  }>

  // Aggregate Stats
  total_tokens_used: number
  total_duration_ms: number

  started_at: string
  completed_at: string | null
  triggered_by: { id: string; display_name: string }
}
```

---

## 🔌 API Routes

```
# ── Templates (CRUD) ──
GET     /api/workflows                           → List templates (filter: workflow_type, category)
POST    /api/workflows                           → Create template
GET     /api/workflows/:id                       → Get template detail
PUT     /api/workflows/:id                       → Update template
DELETE  /api/workflows/:id                       → Delete template
POST    /api/workflows/:id/clone                 → Clone template

# ── Execution ──
POST    /api/workflows/:id/run                   → Run workflow (create instance)
POST    /api/workflows/instances/:id/cancel       → Cancel running instance
POST    /api/workflows/instances/:id/retry        → Retry failed instance

# ── Human-in-the-loop ──
GET     /api/workflows/approvals                 → List pending approvals (for current user)
POST    /api/workflows/instances/:id/steps/:step/approve  → Approve/reject step
GET     /api/workflows/instances/:id/steps/:step/approval → Get approval detail

# ── Instances (History) ──
GET     /api/workflows/:id/instances             → List instances of a template
GET     /api/workflows/instances/:id             → Get instance detail (full trace)
GET     /api/workflows/instances/:id/steps/:step → Get step detail

# ── Analytics ──
GET     /api/workflows/:id/stats                 → Template usage stats

# ── Event-driven ──
POST    /api/workflows/webhooks/:id              → Receive webhook trigger
```

---

## 🧩 Service Layer

### WorkflowEngineService

```typescript
export class WorkflowEngineService {
  /**
   * Run workflow: Tạo instance → thực thi theo workflow_type
   */
  async runWorkflow(templateId: string, inputData: Record<string, any>, userId: string): Promise<WorkflowInstance>

  /**
   * Execute: Chạy từng step tuần tự, xử lý branching + approval gates
   * 
   * Flow:
   * 1. Load WorkflowTemplate + validate input
   * 2. Create WorkflowInstance (status: 'running')
   * 3. Build ExecutionContext = { workflow_input, step_results: {} }
   * 4. For each step:
   *    a. Evaluate conditions → skip if not met
   *    b. Check step_type:
   *       - 'agent_call' → resolve input → load agent → call LLM → save result
   *       - 'condition' → evaluate → determine next step (branching)
   *       - 'approval_gate' → pause instance → notify approvers → wait
   *       - 'delay' → wait configured duration
   *       - 'webhook' → call external URL
   *    c. Update ExecutionContext with step result
   * 5. Update Instance status + aggregate stats
   */
  private async execute(instanceId: string): Promise<void>

  /**
   * Resume workflow after approval decision
   */
  async resumeAfterApproval(instanceId: string, stepOrder: number, decision: ApprovalDecision): Promise<void>

  /**
   * Evaluate conditions: Kiểm tra xem step có nên chạy không
   */
  private evaluateConditions(conditions: Condition[], context: ExecutionContext): boolean

  /**
   * Resolve branching: Determine next step based on condition result
   */
  private resolveNextStep(currentStep: WorkflowStep, conditionResult: boolean, allSteps: WorkflowStep[]): WorkflowStep | null

  /**
   * Resolve input: Lấy input cho step từ previous output hoặc workflow input
   */
  private resolveInput(mapping: InputMapping, context: ExecutionContext): string

  /**
   * Render prompt template với variables
   */
  private renderPrompt(template: string, variables: Record<string, any>): string
}
```

### WorkflowTriggerService (Event-driven)

```typescript
export class WorkflowTriggerService {
  /**
   * Register event listeners on startup
   * Scan all templates with trigger_config.type = 'event'
   * Subscribe to relevant events
   */
  async registerEventListeners(): Promise<void>

  /**
   * Handle incoming event → find matching templates → run workflows
   */
  async handleEvent(eventType: string, eventData: Record<string, any>): Promise<void>

  /**
   * Handle scheduled trigger (called by cron job)
   */
  async handleScheduledTrigger(templateId: string): Promise<void>

  /**
   * Handle webhook trigger
   * Validate webhook secret → extract payload → run workflow
   */
  async handleWebhook(webhookId: string, payload: Record<string, any>): Promise<void>
}
```

---

## 💡 Template Examples

### Example 1: Linear — Meeting Summary → Action Items

```json
{
  "name": "Meeting to Actions",
  "workflow_type": "linear",
  "trigger_config": { "type": "manual" },
  "input_schema": {
    "fields": [
      { "key": "transcript", "type": "textarea", "label": "Meeting Transcript", "required": true },
      { "key": "project_id", "type": "project_id", "label": "Project", "required": true }
    ]
  },
  "steps_config": [
    {
      "step_order": 1,
      "name": "Summarize Meeting",
      "step_type": "agent_call",
      "agent_id": "writer-agent",
      "model_override": { "provider": "openai", "model": "gpt-4o" },
      "prompt_template": "Tóm tắt cuộc họp sau thành bullet points:\n\n{{transcript}}",
      "input_mapping": { "type": "workflow_input", "source": "transcript" }
    },
    {
      "step_order": 2,
      "name": "Extract Action Items",
      "step_type": "agent_call",
      "agent_id": "pm-agent",
      "prompt_template": "Từ tóm tắt, trích xuất action items:\n\n{{input}}\n\nFormat: JSON [{title, assignee_hint, priority}]",
      "input_mapping": { "type": "previous_step", "source": "step_1.output" }
    }
  ]
}
```

### Example 2: Conditional — Bug Triage

```json
{
  "name": "Bug Triage",
  "workflow_type": "conditional",
  "trigger_config": { "type": "event", "event": "issue_created" },
  "steps_config": [
    {
      "step_order": 1,
      "name": "Analyze Bug Severity",
      "step_type": "agent_call",
      "agent_id": "analyst-agent",
      "prompt_template": "Analyze this bug report and return JSON { severity: 'critical'|'high'|'medium'|'low', category: string }:\n\n{{input}}"
    },
    {
      "step_order": 2,
      "name": "Check Severity",
      "step_type": "condition",
      "conditions": [{ "field": "step_1.output.severity", "operator": "equals", "value": "critical" }],
      "branch_config": { "on_true": "step_3", "on_false": "step_4" }
    },
    {
      "step_order": 3,
      "name": "Escalate Critical Bug",
      "step_type": "agent_call",
      "agent_id": "pm-agent",
      "prompt_template": "CRITICAL BUG detected. Draft escalation notification:\n\n{{previous}}"
    },
    {
      "step_order": 4,
      "name": "Standard Assignment",
      "step_type": "agent_call",
      "agent_id": "pm-agent",
      "prompt_template": "Assign this bug to appropriate team member:\n\n{{input}}"
    }
  ]
}
```

### Example 3: Human-in-the-loop — Auto-generate Issues

```json
{
  "name": "Spec to Issues (with Review)",
  "workflow_type": "human_in_the_loop",
  "trigger_config": { "type": "manual" },
  "steps_config": [
    {
      "step_order": 1,
      "name": "Analyze Spec",
      "step_type": "agent_call",
      "agent_id": "analyst-agent",
      "prompt_template": "Analyze spec and extract requirements:\n\n{{input}}"
    },
    {
      "step_order": 2,
      "name": "Generate Issues",
      "step_type": "agent_call",
      "agent_id": "pm-agent",
      "prompt_template": "Convert requirements to issue list:\n\n{{previous}}"
    },
    {
      "step_order": 3,
      "name": "PM Review",
      "step_type": "approval_gate",
      "approval_config": {
        "approver_roles": ["admin", "pm"],
        "timeout_ms": 86400000,
        "timeout_action": "auto_cancel",
        "message_template": "Please review these AI-generated issues before they are created in the project"
      }
    },
    {
      "step_order": 4,
      "name": "Create Issues in Project",
      "step_type": "agent_call",
      "agent_id": "dev-agent",
      "prompt_template": "Create approved issues in project:\n\n{{previous}}"
    }
  ]
}
```

---

## 📊 Traceability & Optimization

### Mọi bước đều truy vết được

Mỗi `WorkflowInstance` lưu chi tiết:

| Field | Mô tả |
|---|---|
| `step.input_preview` | Input gửi vào step (truncated) |
| `step.output` | Output đầy đủ |
| `step.tokens_used` | Số tokens tiêu thụ |
| `step.duration_ms` | Thời gian thực thi |
| `step.model_used` | Model thực tế đã dùng |
| `step.error` | Lỗi nếu có |
| `step.approval` | Approval decision + comment (HITL) |

### Tối ưu hóa workflow

Từ dữ liệu truy vết, người dùng có thể:

1. **So sánh** kết quả giữa các lần chạy (compare instances)
2. **Thay đổi model** ở từng step (ví dụ: GPT-4o → Gemini cho step rẻ hơn)
3. **Điều chỉnh prompt** dựa trên output thực tế
4. **Thêm/bớt steps** để cải thiện kết quả
5. **Thêm approval gate** tại step quan trọng

---

## 🛡️ Security Considerations

- ✅ **Permission check** — Chỉ user có quyền mới được tạo/chạy workflow
- ✅ **Agent validation** — Verify tất cả agent_ids trong steps_config tồn tại và active
- ✅ **Execution timeout** — Giới hạn tổng thời gian chạy workflow (default: 10 phút)
- ✅ **Token budget** — Cấu hình max tokens per workflow run
- ✅ **Idempotency** — Mỗi instance có unique ID, safe to retry
- ✅ **Cancellation** — User có thể cancel workflow đang chạy
- ✅ **Webhook security** — Validate secret/signature trước khi trigger
- ✅ **Approval auth** — Chỉ user có role trong `approver_roles` mới approve được

---

## 📚 Related Documents

- [08-agents-module.md](08-agents-module.md) — Agent definitions
- [09-chat-module.md](09-chat-module.md) — Chat UI integration
- [../../services/02-ai-gateway.md](../../services/02-ai-gateway.md) — LLM routing
- [../../services/03-ai-tools.md](../../services/03-ai-tools.md) — Tool definitions
- [../../database/tables/workflow_templates.md](../../database/tables/workflow_templates.md) — DB schema
- [../../database/tables/workflow_instances.md](../../database/tables/workflow_instances.md) — Execution history

---

*Last Updated: 2026-02-25 — v3 (Added Workflow classification, Conditional branching, Event-driven triggers, Human-in-the-loop approval gates)*
