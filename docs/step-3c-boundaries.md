# Step 3c: Xác định Boundaries

**Date:** 2026-02-07  
**Status:** ✅ Completed

## Sources

- Previous: [Step 3 - High-Level Architecture](step-3-high-level-architecture.md)
- [Step 3b - Data Flows](step-3b-data-flows.md)
- Discord: Message 1469602954221719699

---

## 1. Architectural Boundaries Overview

```
┌─────────────────────────────────────────────────────────────┐
│                     SYSTEM BOUNDARIES                        │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│   ┌───────────────┐                                         │
│   │   FRONTEND    │  📍 UNTRUSTED ZONE                      │
│   │   (Next.js)   │                                         │
│   └───────┬───────┘                                         │
│           │                                                 │
│           │  API Calls (JWT + tenant_id)                    │
│           ▼                                                 │
│   ┌───────────────┐                                         │
│   │    BACKEND    │  📍 SOURCE OF TRUTH                     │
│   │   (Node.js)   │  ✅ Auth, Rules, Orchestration          │
│   └───────┬───────┘                                         │
│           │                                                 │
│           │  Controlled AI Calls                            │
│           ▼                                                 │
│   ┌───────────────┐                                         │
│ │       AI        │  📍 REASONING ENGINE                    │
│   │  (LLM Proxy)  │  ✅ Suggestions, Patterns               │
│   └───────────────┘                                         │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## 2. Frontend Boundary (UNTRUSTED ZONE)

### 2.1 What Frontend CAN Do

| ✅ Allowed | Description |
|------------|-------------|
| UI Rendering | Hiển thị components, forms, dashboards |
| Input Collection | Gửi user input lên backend |
| Output Display | Nhận và hiển thị response từ backend |
| UI State Management | Quản lý local state (loading, form data) |

### 2.2 What Frontend CANNOT Do

| ❌ Forbidden | Reason |
|-------------|--------|
| Direct LLM Calls | Security - secrets có thể bị leak |
| Workflow Decisions | Backend phải là source of truth |
| Store Secrets | API keys không được lưu ở frontend |
| Permission Checks | Backend phải validate quyền |

### 2.3 Frontend Security Rules

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

---

## 3. Backend Boundary (SOURCE OF TRUTH)

### 3.1 Backend Responsibilities

| ✅ Responsibility | Description |
|-----------------|-------------|
| **Auth / Tenant Isolation** | Validate JWT, enforce tenant boundaries |
| **Business Rules** | Enforce domain logic, validation |
| **Workflow Orchestration** | Manage workflow state machine |
| **Tool Permission** | Control what AI agents can do |
| **Cost / Quota** | Track và limit AI usage |
| **Audit / Log** | Record all important actions |

### 3.2 Backend Architecture

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

### 3.3 Backend as Source of Truth

```
┌─────────────────────────────────────────────────────────┐
│  BACKEND IS SOURCE OF TRUTH                             │
├─────────────────────────────────────────────────────────┤
│  ✅ Mọi quyết định phải qua backend                     │
│  ✅ Frontend chỉ là "dumb terminal"                     │
│  ✅ AI suggestions PHẢI được backend validate           │
│  ✅ User permissions PHẢI được backend check            │
│  ✅ AI resource access PHẢI qua backend                 │
└─────────────────────────────────────────────────────────┘
```

---

## 4. AI Boundary (AI là gì trong hệ thống?)

### 4.1 AI Role: NOT a Decision Maker

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

### 4.2 What AI CAN Do

| ✅ AI Capabilities | Description |
|-------------------|-------------|
| **Reasoning Engine** | Phân tích, suy luận từ context |
| **Suggestion Engine** | Đề xuất hành động, giải pháp |
| **Pattern Matcher** | Tìm patterns trong data |
| **Text Generation** | Sinh text, documentation, summaries |
| **Query Memory** | Truy vấn knowledge base (qua backend) |

### 4.3 What AI CANNOT Do

| ❌ AI Forbidden | Reason |
|-----------------|--------|
| **Self-Decision Making** | Mọi action phải qua backend approval |
| **Direct DB Access** | Không được query DB trực tiếp |
| **Uncontrolled API Calls** | Mọi call phải qua tool permission system |
| **Cross-Tenant Knowledge** | Không được biết về tenant khác |
| **Secret Exposure** | Không được expose secrets trong response |

### 4.4 AI Communication Flow

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

**Key Points:**
1. Frontend → Backend: Send request
2. Backend → AI: Proxy với tenant context
3. AI → Backend: Return suggestions
4. Backend → Frontend: Validated response

---

## 5. Boundaries Summary

### 5.1 Quick Reference

| Boundary | Zone | Trust Level | Primary Role |
|----------|------|-------------|--------------|
| **Frontend** | Untrusted | 🔴 Low | UI only |
| **Backend** | Source of Truth | 🟢 High | Control everything |
| **AI** | Reasoning Engine | 🟡 Medium | Suggest only |

### 5.2 Decision Flow

```
User Action
    │
    ▼
Frontend (UI Only)
    │
    ▼
Backend (All Decisions)
    │
    ├──── Needs AI? ──▶ AI (Suggestions)
    │                      │
    │                      ▼
    │                Backend (Approve/Deny)
    │
    ▼
Frontend (Display Result)
```

---

## 6. Security Implications

### 6.1 Tenant Isolation Enforced By

| Layer | Enforcement |
|-------|-------------|
| **Frontend** | Cannot prevent bypass - UNTRUSTED |
| **Backend** | ✅ Required - All requests validated |
| **AI** | ✅ Required - Tenant context in every prompt |

### 6.2 Defense in Depth

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

## 7. Related Documents

- [Step 3 - High-Level Architecture](step-3-high-level-architecture.md)
- [Step 3b - Data Flows](step-3b-data-flows.md)
- [Step 2 - MVP Scope](step-2-mvp-scope.md)

---

## 8. GitHub Issues

- #1: Tạo repo và lập kế hoạch
- #2: Step 2 - MVP Scope Clarification
- #3: Step 3 - High-Level Architecture
- #4: Step 3b - Data Flows
- #5: Step 3c - Boundaries: Frontend / Backend / AI

---

*Document Version: 1.0*
*Last Updated: 2026-02-07*
