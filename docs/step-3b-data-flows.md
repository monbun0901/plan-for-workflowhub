# Step 3b: Data Flows cho Use Cases Chính

**Date:** 2026-02-07  
**Status:** ✅ Completed

## Sources

- Previous: [Step 3 - High-Level Architecture](step-3-high-level-architecture.md)
- Discord: Message 1469602377865756683

---

## 1. Use Case: User → AI Response

### Data Flow Diagram

```
┌──────────────┐     ┌──────────────────────┐     ┌─────────────────┐
│   Frontend   │────▶│  API Gateway /      │────▶│  AI Orchestrator │
│  (Next.js)   │     │  Backend (Node.js)  │     │                 │
└──────────────┘     └──────────────────────┘     └────────┬────────┘
       ▲                                               │
       │                                               │
       │              ┌──────────────────────┐        │
       │              │  Vector Store        │◀───────┤
       │              │  (RAG Memory)        │        │
       │              └──────────────────────┘        │
       │                                               │
       │              ┌──────────────────────┐        │
       │              │   LLM Provider       │───────┘
       │              │  OpenAI / Anthropic  │        │
       └──────────────│                      │◀───────┘
                      └──────────────────────┘
```

### Step-by-Step Flow

| Step | Component | Action | Tenant Check |
|------|-----------|--------|--------------|
| 1 | Frontend | send message | JWT + tenant_id |
| 2 | API Gateway | auth + tenant check | ✅ Required |
| 3 | Backend | save message | ✅ Required |
| 4 | AI Orchestrator | retrieve memory (vector DB) | ✅ **tenant filter** |
| 5 | AI Orchestrator | build prompt | - |
| 6 | AI Orchestrator | call LLM | - |
| 7 | LLM Provider | response | - |
| 8 | AI Orchestrator | post-process / guardrails | - |
| 9 | Backend | store memory | ✅ **tenant isolation** |
| 10 | Backend → Frontend | response | - |

---

## 2. Use Case: Workflow Trigger (AI → Action)

### Data Flow Diagram

```
┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐
│   AI Output     │────▶│  Rule Engine    │────▶│ Workflow Engine │
│                 │     │                 │     │                 │
└─────────────────┘     └─────────────────┘     └────────┬────────┘
                                                      │
                                                      ▼
                              ┌─────────────────┐     ┌─────────────────┐
                              │   LLM Provider   │     │Action Executor │
                              │  (Intent Detect)│     │                 │
                              └─────────────────┘     └─────────────────┘
```

### Key Rule

```
┌─────────────────────────────────────────────────┐
│  ⚠️  AI KHÔNG ĐƯỢC TRIGGER ACTION TRỰC TIẾP   │
└─────────────────────────────────────────────────┘

AI Output ──▶ Rule Engine ──▶ Workflow Engine ──▶ Action Executor
     │             │                │                  │
     │         Check          State Machine        Execute
     │         Condition      Idempotency          Action
     ▼             ▼               ▼                  ▼
  Intent       Condition      Transition         Final
  Detect       Evaluate       Validated          Result
```

### Step-by-Step Flow

| Step | Component | Action | Purpose |
|------|-----------|--------|---------|
| 1 | AI Output | intent detected | Parse user intent |
| 2 | Rule Engine | condition check | Validate rules |
| 3 | Workflow Engine | state machine | Manage workflow state |
| 4 | Workflow Engine | idempotency check | Prevent duplicate |
| 5 | Action Executor | execute action | Final action |

---

## 3. Edge Points (Điểm dễ lỗi)

### Critical Edge Cases

| Step | Rủi ro | Hậu quả | Xử lý |
|------|--------|---------|-------|
| **4** | Quên filter tenant | Data leak giữa organizations | Bắt buộc `tenant_id` trong mọi query |
| **6** | Agent gọi tool vượt quyền | Security breach | Sandbox + permission check |
| **9** | Lưu memory sai scope | Data corruption | Tenant isolation validation |

### Validation Rules

```typescript
// Pseudocode cho edge case handling

// Step 4: Tenant filter bắt buộc
async function retrieveMemory(query: string, tenantId: string) {
  if (!tenantId) throw new Error('tenant_id required');
  return vectorDB.query(query, { filter: { tenant_id: tenantId } });
}

// Step 6: Sandbox + permission check
async function agentCallTool(toolName: string, params: any) {
  const permissions = getAgentPermissions(agentId);
  if (!permissions.includes(toolName)) {
    throw new Error('Tool not allowed');
  }
  return sandboxedExecute(toolName, params);
}

// Step 9: Tenant isolation
async function storeMemory(memory: Memory, tenantId: string) {
  const validatedMemory = { ...memory, tenant_id: tenantId };
  return vectorDB.insert(validatedMemory);
}
```

---

## 4. Security & Safety

### Multi-Tenant Isolation

```
┌─────────────────────────────────────────────────────────┐
│                    TENANT ISOLATION RULES               │
├─────────────────────────────────────────────────────────┤
│  ✅ Mọi query PHẢI có tenant_id                         │
│  ✅ Memory PHẢI được tag với tenant_id                  │
│  ✅ Agent CHỈ được truy cập data trong tenant của nó   │
│  ✅ AI Response PHẢI được filter theo tenant             │
│  ❌ KHÔNG CHO phép cross-tenant access                  │
└─────────────────────────────────────────────────────────┘
```

### AI Safety

```
┌─────────────────────────────────────────────────────────┐
│                    AI SAFETY RULES                       │
├─────────────────────────────────────────────────────────┤
│  ✅ Post-process response trước khi gửi về frontend     │
│  ✅ Guardrails để lọc harmful content                   │
│  ✅ Intent detection trước khi trigger workflow        │
│  ✅ Idempotency check để tránh duplicate actions       │
│  ❌ AI KHÔNG được execute action trực tiếp             │
└─────────────────────────────────────────────────────────┘
```

---

## 5. Related Documents

- [Step 3 - High-Level Architecture](step-3-high-level-architecture.md)
- [Step 2 - MVP Scope](step-2-mvp-scope.md)

---

## 6. GitHub Issues

- #1: Tạo repo và lập kế hoạch
- #2: Step 2 - MVP Scope Clarification
- #3: Step 3 - High-Level Architecture
- #4: Step 3b - Data Flows cho Use Cases chính

---

*Document Version: 1.0*
*Last Updated: 2026-02-07*
