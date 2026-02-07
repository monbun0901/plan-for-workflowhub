# Step 3: High-Level Architecture - COMPLETE

**Date:** 2026-02-07  
**Status:** ✅ Completed

## Overview

Tài liệu này tổng hợp toàn bộ Step 3 - Thiết kế kiến trúc High-Level.

---

## 1. Architecture High-Level (Summary)

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

## 2. Components Description

### 2.1 Frontend (Untrusted Zone)
- **Role:** UI rendering, input collection, output display
- **Boundary:** Cannot call LLM directly, cannot make decisions
- **Reference:** [Step 3c - Boundaries](step-3c-boundaries.md)

### 2.2 Backend API (Source of Truth)
| Module | Responsibility |
|--------|----------------|
| **Auth / Tenant** | JWT validation, tenant isolation |
| **Business Logic** | Domain rules, validation |
| **Workflow Engine** | State machine, orchestration |
| **AI Orchestrator** | Prompt building, guardrails, tool routing |

### 2.3 Data Layer
| Database | Purpose |
|----------|---------|
| **Vector DB** | RAG memory, embeddings, semantic search |
| **Relational DB (MySQL)** | Users, organizations, projects, issues, tasks |

---

## 3. Data Flows

### 3.1 User → AI Response Flow
```
Frontend → Backend → AI Orchestrator → LLM → Backend → Frontend
```
**Key Points:**
- Tenant filter trong mọi vector DB query
- AI KHÔNG được trigger action trực tiếp

**Reference:** [Step 3b - Data Flows](step-3b-data-flows.md)

### 3.2 Workflow Trigger Flow
```
AI Output → Rule Engine → Workflow Engine → Action Executor
```
**Key Points:**
- AI detects intent → Rule validates → Workflow executes
- Idempotency check

---

## 4. Boundaries Summary

| Layer | Trust Level | Cannot Do |
|-------|-------------|-----------|
| **Frontend** | 🔴 Low | Direct LLM, secrets, decisions |
| **Backend** | 🟢 High | Everything controlled |
| **AI** | 🟡 Medium | Direct actions, DB access |

**Reference:** [Step 3c - Boundaries](step-3c-boundaries.md)

---

## 5. Nguyên Tắc Vàng (GOLDEN RULES)

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

## 6. Decision Tree

```
Decision Point
      │
      ├─► AI involved?
      │       │
      │       ├─ YES → Backend validates → Frontend displays
      │       │
      │       └─ NO  → Backend decides
      │
      └─► Action needed?
              │
              ├─ YES → Rule Engine → Workflow → Execute
              │
              └─ NO  → Response only
```

---

## 7. Documents in Step 3

| Doc | Content | Status |
|-----|---------|--------|
| [step-3-high-level-architecture.md](step-3-high-level-architecture.md) | Modular Monolith, 5 core modules | ✅ |
| [step-3b-data-flows.md](step-3b-data-flows.md) | User→AI, AI→Workflow flows + edge cases | ✅ |
| [step-3c-boundaries.md](step-3c-boundaries.md) | Frontend/Backend/AI boundaries | ✅ |
| [step-3d-architecture-summary.md](step-3d-architecture-summary.md) | This document | ✅ |

---

## 8. GitHub Issues

- #1: Tạo repo và lập kế hoạch
- #2: Step 2 - MVP Scope Clarification
- #3: Step 3 - High-Level Architecture
- #4: Step 3b - Data Flows
- #5: Step 3c - Boundaries
- #6: Step 3d - Architecture Summary & Golden Rules

---

## 9. Next Steps

Step 3 hoàn thành. Chờ Boss chỉ định Step tiếp theo.

---

*Document Version: 1.0*
*Last Updated: 2026-02-07*
