# Step 3: High-Level Architecture

**Date:** 2026-02-07  
**Status:** ✅ Completed

## Sources

- Previous: [Step 2 - MVP Scope](step-2-mvp-scope.md)
- Discord: Message 1469601714285056070

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

## 3. Boundaries Strategy

### Code Boundaries (NOT Network)

```
src/
├── modules/
│   ├── auth/           #独立的 authentication 模块
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

## 4. Tech Stack Integration

| Layer | Tech | Module |
|-------|------|--------|
| **Frontend** | Next.js | UI layer |
| **Backend** | Node.js | All modules |
| **Database** | MySQL | Primary DB |
| **Vector DB** | Pinecone/Weaviate | vector-store |
| **AI** | OpenAI/Anthropic | ai-agent |

---

## 5. Migration Path (Tương lai)

```
Modular Monolith → ? microservices khi cần
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

**Nguyên tắc:** Migrate khi THỰC SỰ CẦN, không phải khi NGHĩ CẦN.

---

## 6. Related Documents

- [Step 2 - MVP Scope](step-2-mvp-scope.md)
- [v1 - Initial Plan](../v1-initial-plan.md)

---

## 7. GitHub Issues

- #1: Tạo repo và lập kế hoạch
- #2: Step 2 - MVP Scope Clarification
- #3: Step 3 - High-Level Architecture

---

*Document Version: 1.0*
*Last Updated: 2026-02-07*
