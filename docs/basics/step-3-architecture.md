# Step 3: Architecture Design

**Date:** 2026-02-07  
**Status:** ✅ Completed

## Sources

- Previous: [Step 2 - MVP Scope](step-2-mvp-scope.md)
- Discord Messages: 1469601714285056070, 1469602377865756683, 1469602954221719699

---

## 🎯 Overview

Thiết kế kiến trúc High-Level cho WorkflowHub:
- **Pattern:** Modular Monolith
- **Core Modules:** 5 modules (auth, tenant, workflow, ai-agent, vector-store)
- **Boundaries:** Frontend/Backend/AI with clear responsibilities
- **Data Flows:** User→AI và AI→Workflow patterns

---

## 1. Architectural Decision

### Chosen: Modular Monolith

**Nguyên tắc chọn kiến trúc:**

| Principle | Description |
|-----------|-------------|
| ❌ No Trends | Không chọn theo xu hướng |
| ✅ By Complexity | Chọn theo độ phức tạp + giai đoạn sản phẩm |
| ✅ MVP First | Phù hợp cho giai đoạn đầu của sản phẩm |

**Why Modular Monolith?**
- Đơn giản hóa deployment
- Dễ dàng refactor sau này khi cần
- Phù hợp với team nhỏ
- Chi phí vận hành thấp

### System Overview

```
┌─────────────────────────────────────────────────────────────┐
│                        WORKFLOWHUB                           │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│   ┌──────────────┐                                          │
│   │   Frontend   │  📍 UNTRUSTED ZONE                      │
│   │   (Next.js)  │                                          │
│   └──────┬───────┘                                          │
│          │                                                   │
│          ▼                                                   │
│   ┌──────────────┐                                          │
│   │  Backend API  │  📍 SOURCE OF TRUTH                    │
│   └──────┬───────┘                                          │
│          │                                                   │
│   ┌──────┴───────────────────────────────────────┐        │
│   │              Backend Modules                  │        │
│   ├──────────────────────────────────────────────┤        │
│   │  ├─ Auth / Tenant         (Multi-tenant)    │        │
│   │  ├─ Business Logic        (Domain rules)    │        │
│   │  ├─ Workflow Engine       (Automation)      │        │
│   │  ├─ AI Orchestrator                        │        │
│   │  │    ├─ Prompt Builder                    │        │
│   │  │    ├─ Guardrails                       │        │
│   │  │    └─ Tool Router                       │        │
│   │  ├─ Vector DB             (RAG Memory)      │        │
│   │  └─ Relational DB         (MySQL)           │        │
│   └──────────────────────────────────────────────┘        │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

---

## 2. Core Modules

```
┌─────────────────────────────────────────────────┐
│              WorkflowHub App                     │
├─────────────────────────────────────────────────┤
│  ┌──────────┐  ┌──────────┐  ┌──────────┐      │
│  │   auth    │  │  tenant  │  │ workflow │      │
│  └──────────┘  └──────────┘  └──────────┘      │
│                                                 │
│  ┌──────────┐  ┌──────────┐                    │
│  │ ai-agent │  │vector-   │                    │
│  │          │  │  store   │                    │
│  └──────────┘  └──────────┘                    │
└─────────────────────────────────────────────────┘
```

### 2.1 Module: auth
**Chức năng:** Authentication & Authorization
- JWT-based authentication
- RBAC/ABAC permissions
- Session management

### 2.2 Module: tenant
**Chức năng:** Multi-tenant isolation
- Organization management
- Tenant context isolation
- Resource boundaries

### 2.3 Module: workflow
**Chức năng:** Workflow automation engine
- Workflow Templates
- Workflow Instances
- Triggers & Conditions

### 2.4 Module: ai-agent
**Chức năng:** AI Agents core
- Agent creation & management
- System prompts
- Agent-Project assignment

### 2.5 Module: vector-store
**Chức năng:** RAG/Vector database
- Document embeddings
- Semantic search
- Knowledge base retrieval

---

## 3. Code Boundaries (NOT Network)

```
src/
├── modules/
│   ├── auth/           # 独立的 authentication 模块
│   ├── tenant/
│   ├── workflow/
│   ├── ai-agent/
│   └── vector-store/
├── shared/
│   ├── database/       # Shared database utilities
│   ├── utils/          # Common utilities
│   └── middleware/     # Shared middleware
└── app.ts              # Composition root
```

**Key Points:**
- ✅ Module isolation bằng **code structure**
- ✅ Clear module boundaries trong package
- ✅ Shared resources được quản lý chặt chẽ
- ❌ NO network boundaries (chưa cần microservices)

---

## 4. Data Flows

### 4.1 User → AI Response Flow

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

**Step-by-Step:**

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

### 4.2 Workflow Trigger (AI → Action) Flow

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

**Key Rule:**

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

**Step-by-Step:**

| Step | Component | Action | Purpose |
|------|-----------|--------|---------|
| 1 | AI Output | intent detected | Parse user intent |
| 2 | Rule Engine | condition check | Validate rules |
| 3 | Workflow Engine | state machine | Manage workflow state |
| 4 | Workflow Engine | idempotency check | Prevent duplicate |
| 5 | Action Executor | execute action | Final action |

### 4.3 Critical Edge Cases

| Step | Rủi ro | Hậu quả | Xử lý |
|------|--------|---------|-------|
| **4** | Quên filter tenant | Data leak giữa organizations | Bắt buộc `tenant_id` trong mọi query |
| **6** | Agent gọi tool vượt quyền | Security breach | Sandbox + permission check |
| **9** | Lưu memory sai scope | Data corruption | Tenant isolation validation |

---

## 5. System Boundaries

### 5.1 Frontend Boundary (UNTRUSTED ZONE)

```
┌─────────────────────────────────────────────────────────┐
│                    FRONTEND ZONE                        │
├─────────────────────────────────────────────────────────┤
│  ✅ CAN DO:                                             │
│     - UI Rendering                                      │
│     - Input Collection                                  │
│     - Output Display                                    │
│     - UI State Management                               │
│                                                          │
│  ❌ CANNOT DO:                                          │
│     - Direct LLM Calls (secrets leak)                  │
│     - Workflow Decisions (backend = source of truth)   │
│     - Store Secrets (API keys)                         │
│     - Permission Checks (backend must validate)        │
└─────────────────────────────────────────────────────────┘
```

**Security Example:**

```typescript
// ❌ WRONG - Never do this
const response = await openai.chat.completions.create({
  apiKey: process.env.NEXT_PUBLIC_OPENAI_KEY  // EXPOSED!
});

// ✅ CORRECT - Always proxy through backend
const response = await fetch('/api/ai/chat', {
  method: 'POST',
  headers: { 'Authorization': `Bearer ${token}` },
  body: JSON.stringify({ message })
});
```

### 5.2 Backend Boundary (SOURCE OF TRUTH)

**Responsibilities:**

| ✅ Responsibility | Description |
|-----------------|-------------|
| **Auth / Tenant Isolation** | Validate JWT, enforce tenant boundaries |
| **Business Rules** | Enforce domain logic, validation |
| **Workflow Orchestration** | Manage workflow state machine |
| **Tool Permission** | Control what AI agents can do |
| **Cost / Quota** | Track và limit AI usage |
| **Audit / Log** | Record all important actions |

**Architecture Layers:**

```
┌─────────────────────────────────────────────────────────┐
│                    BACKEND LAYERS                        │
├─────────────────────────────────────────────────────────┤
│  ┌─────────────────────────────────────────────────┐    │
│  │           API Layer (REST / GraphQL)            │    │
│  └─────────────────────────────────────────────────┘    │
│                          │                               │
│  ┌─────────────────────────────────────────────────┐    │
│  │         Business Logic / Services                │    │
│  │  ┌─────────┐ ┌─────────┐ ┌─────────────────┐   │    │
│  │  │  Auth   │ │ Tenant  │ │  Workflow       │   │    │
│  │  └─────────┘ └─────────┘ │  Orchestrator   │   │    │
│  │                         └─────────────────┘   │    │
│  └─────────────────────────────────────────────────┘    │
│                          │                               │
│  ┌─────────────────────────────────────────────────┐    │
│  │           Data Access / Repositories             │    │
│  └─────────────────────────────────────────────────┘    │
│                          │                               │
│  ┌─────────────────────────────────────────────────┐    │
│  │              Database (MySQL)                     │    │
│  └─────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────┘
```

### 5.3 AI Boundary (REASONING ENGINE)

**AI Role: NOT a Decision Maker**

```
┌─────────────────────────────────────────────────────────┐
│           AI IS NOT A DECISION MAKER                   │
├─────────────────────────────────────────────────────────┤
│                                                         │
│   User Request                                          │
│        │                                                │
│        ▼                                                │
│   ┌──────────────┐                                      │
│   │      AI      │  ❌ Cannot make decisions              │
│   │  (LLM Only)  │     ❌ Cannot execute actions          │
│   └──────┬───────┘                                      │
│          │                                              │
│          ▼                                              │
│   ┌──────────────┐                                      │
│   │   Backend    │  ✅ Makes final decisions              │
│   │ (Enforcer)   │     ✅ Controls all actions           │
│   └──────────────┘                                      │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

**What AI CAN Do:**

| ✅ AI Capabilities | Description |
|-------------------|-------------|
| **Reasoning Engine** | Phân tích, suy luận từ context |
| **Suggestion Engine** | Đề xuất hành động, giải pháp |
| **Pattern Matcher** | Tìm patterns trong data |
| **Text Generation** | Sinh text, documentation, summaries |
| **Query Memory** | Truy vấn knowledge base (qua backend) |

**What AI CANNOT Do:**

| ❌ AI Forbidden | Reason |
|-----------------|--------|
| **Self-Decision Making** | Mọi action phải qua backend approval |
| **Direct DB Access** | Không được query DB trực tiếp |
| **Uncontrolled API Calls** | Mọi call phải qua tool permission system |
| **Cross-Tenant Knowledge** | Không được biết về tenant khác |
| **Secret Exposure** | Không được expose secrets trong response |

**Communication Flow:**

```
┌──────────┐     ┌──────────┐     ┌──────────┐     ┌──────────┐
│  Frontend│────▶│  Backend │────▶│   AI     │────▶│  Backend │
│          │     │ (Proxy)  │     │ (LLM)    │     │(Validate)│
└──────────┘     └──────────┘     └──────────┘     └────┬─────┘
                                                       │
                                                       ▼
                                                ┌──────────┐
                                                │ Frontend │
                                                │(Response)│
                                                └──────────┘
```

### 5.4 Boundaries Summary

| Boundary | Zone | Trust Level | Primary Role |
|----------|------|-------------|--------------|
| **Frontend** | Untrusted | 🔴 Low | UI only |
| **Backend** | Source of Truth | 🟢 High | Control everything |
| **AI** | Reasoning Engine | 🟡 Medium | Suggest only |

---

## 6. Security & Multi-Tenant Isolation

### 6.1 Tenant Isolation Rules

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

### 6.2 AI Safety Rules

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

### 6.3 Defense in Depth

```
┌─────────────────────────────────────────────────────────┐
│              DEFENSE IN DEPTH                           │
├─────────────────────────────────────────────────────────┤
│  Layer 1: Frontend - Basic UI isolation                 │
│  Layer 2: Backend - Auth + Tenant validation            │
│  Layer 3: Database - Row-level security                 │
│  Layer 4: AI - Context filtering                        │
│  Layer 5: Audit - All actions logged                   │
└─────────────────────────────────────────────────────────┘
```

---

## 7. Tech Stack Integration

| Layer | Tech | Module |
|-------|------|--------|
| **Frontend** | Next.js | UI layer |
| **Backend** | Node.js | All modules |
| **Database** | MySQL | Primary DB |
| **Vector DB** | Pinecone/Weaviate/Chroma | vector-store |
| **AI** | OpenAI/Anthropic | ai-agent |

---

## 8. Nguyên Tắc Vàng (GOLDEN RULES)

```
┌─────────────────────────────────────────────────────────────┐
│              🏆 NGUYÊN TẮC VÀNG 🏆                          │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  1️⃣  AI không bao giờ là root of trust                     │
│      → Backend LUÔN là source of truth                      │
│      → AI chỉ là suggestion engine                          │
│                                                              │
│  2️⃣  Tenant boundary > AI intelligence                      │
│      → Multi-tenant isolation PHẢI được enforce             │
│      → AI không được biết về tenant khác                    │
│                                                              │
│  3️⃣  Workflow quyết định bằng rule, không bằng LLM         │
│      → Rule-based decisions                                 │
│      → LLM chỉ để reasoning, không để decide               │
│                                                              │
│  4️⃣  Monolith trước, microservices sau                     │
│      → Modular Monolith với code boundaries                 │
│      → Scale khi THỰC SỰ CẦN, không phải khi NGHĨ CẦN      │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

---

## 9. Migration Path (Tương lai)

```
Modular Monolith → microservices khi cần
                           │
                           ▼
                  ┌────────────────┐
                  │  Khi nào cần?  │
                  ├────────────────┤
                  │ • Team > 10    │
                  │ • Scale cực lớn│
                  │ • Team split   │
                  └────────────────┘
```

**Nguyên tắc:** Migrate khi THỰC SỰ CẦN, không phải khi NGHĨ CẦN.

---

## 10. Related Documents

- [Step 2 - MVP Scope](step-2-mvp-scope.md)
- [Step 4 - Directory Structure](step-4-directory-structure.md)
- [v1 - Initial Plan](v1-initial-plan.md)

---

*Document Version: 2.0 (Consolidated)*  
*Last Updated: 2026-02-11*
