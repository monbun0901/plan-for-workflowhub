# Backend Implementation Guide

**Version:** v2  
**Date:** 2026-02-13  
**Architecture:** Modular Monolith (Controller → Service → Repository)

---

## 📂 Directory Structure

```
backend/
├── core/                 ← Foundation & Cross-cutting
│   ├── 01-architecture.md
│   ├── 02-authentication.md
│   ├── 03-multi-tenant.md
│   └── 04-security-checklist.md
│
├── modules/              ← Feature Modules (10 modules)
│   ├── README.md         ← Module index + dependency graph
│   ├── 01-organizations-module.md
│   ├── 02-members-module.md
│   ├── 03-projects-module.md
│   ├── 04-issues-module.md
│   ├── 05-tasks-module.md
│   ├── 06-documents-module.md
│   ├── 07-master-data-module.md
│   ├── 08-agents-module.md
│   ├── 09-chat-module.md
│   └── 10-workflows-module.md
│
├── dtos/                 ← DTO Specifications (8 entities)
│   ├── README.md
│   ├── 01-projects-dtos.md
│   ├── 02-tasks-dtos.md
│   ├── 03-issues-dtos.md
│   ├── 04-documents-dtos.md
│   ├── 05-organizations-dtos.md
│   ├── 06-members-dtos.md
│   ├── 07-chats-dtos.md
│   └── 08-agents-dtos.md
│
├── services/             ← AI & Infrastructure Services
│   ├── 01-rag-service.md
│   ├── 02-ai-gateway.md
│   └── 03-ai-tools.md
│
├── api-routes-map.md     ← Full API Blueprint
└── README.md             ← This file
```

---

## 🧭 Navigation

### 🟢 Phase 1: Core System (MVP)

**Start here** → Build in this order:

| Step | Document | Description |
|------|----------|-------------|
| 1 | [core/01-architecture.md](core/01-architecture.md) | Layered architecture & folder structure |
| 2 | [core/02-authentication.md](core/02-authentication.md) | JWT, bcrypt & auth flow |
| 3 | [core/03-multi-tenant.md](core/03-multi-tenant.md) | Organization isolation pattern |
| 4 | [core/04-security-checklist.md](core/04-security-checklist.md) | Security audit checklist |
| 5 | [modules/01-organizations-module.md](modules/01-organizations-module.md) | Tenant root CRUD |
| 6 | [modules/02-members-module.md](modules/02-members-module.md) | Membership & roles |
| 7 | [modules/03-projects-module.md](modules/03-projects-module.md) | **Reference CRUD** (study this first) |
| 8 | [modules/04-issues-module.md](modules/04-issues-module.md) | Issue management |
| 9 | [modules/05-tasks-module.md](modules/05-tasks-module.md) | Task management & assignments |
| 10 | [modules/06-documents-module.md](modules/06-documents-module.md) | Knowledge base |
| 11 | [modules/07-master-data-module.md](modules/07-master-data-module.md) | Tags, categories, statuses |

### 🚀 Phase 2: AI & Scale-up

| Step | Document | Description |
|------|----------|-------------|
| 12 | [modules/08-agents-module.md](modules/08-agents-module.md) | AI agent personas |
| 13 | [modules/09-chat-module.md](modules/09-chat-module.md) | AI chat interface |
| 14 | [modules/10-workflows-module.md](modules/10-workflows-module.md) | Automation & state machine |
| 15 | [services/01-rag-service.md](services/01-rag-service.md) | Knowledge engine (RAG) |
| 16 | [services/02-ai-gateway.md](services/02-ai-gateway.md) | AI orchestration |
| 17 | [services/03-ai-tools.md](services/03-ai-tools.md) | AI tools specification |

### 📡 API Reference

| Document | Description |
|----------|-------------|
| [api-routes-map.md](api-routes-map.md) | Full API blueprint (all endpoints) |
| [dtos/README.md](dtos/README.md) | DTO specs index |

---

## 🎯 Quick Start

### For New Developers
1. Read [core/01-architecture.md](core/01-architecture.md) - understand layers
2. Read [core/02-authentication.md](core/02-authentication.md) - JWT setup
3. Study [modules/03-projects-module.md](modules/03-projects-module.md) - reference CRUD
4. Apply same pattern to other modules

### For Security Review
1. [core/02-authentication.md](core/02-authentication.md) → Auth implementation
2. [core/03-multi-tenant.md](core/03-multi-tenant.md) → Tenant isolation
3. [core/04-security-checklist.md](core/04-security-checklist.md) → Security audit

---

## 📐 Module Pattern

```
apps/api/src/modules/{feature}/
├── controllers/
│   └── {feature}.controller.ts    # HTTP handling
├── services/
│   └── {feature}.service.ts       # Business logic
├── repositories/
│   └── {feature}.repository.ts    # Data access
├── models/
│   └── {feature}.model.ts         # Sequelize model
├── dtos/
│   ├── create-{feature}.dto.ts    # Zod validation
│   └── update-{feature}.dto.ts
├── routes/
│   └── {feature}.routes.ts
└── index.ts
```

**Flow:** `Request → Controller → Service → Repository → Database`

---

## 📝 Mandatory Rules

- **All modules** MUST implement tenant isolation (`organization_id` filter)
- **All endpoints** MUST validate input via Zod schemas
- **All services** MUST log important actions
- **All repositories** MUST check tenant ownership before update/delete

---

## 📚 Related Documents

- [../database/README.md](../database/README.md) - Database schemas
- [../common/setup-and-workflows.md](../common/setup-and-workflows.md) - Project setup
- [../database/database-security.md](../database/database-security.md) - Database security

---

*Last Updated: 2026-02-13*
