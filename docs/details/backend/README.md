# Backend Implementation Guide

**Version:** v1  
**Date:** 2026-02-11  
**Skills:** `nodejs-best-practices`, `backend-architect`, `backend-security-coder`

---

## 📋 Overview

Backend implementation guide cho WorkflowHub API sử dụng Express.js, Sequelize, và security best practices.

**Architecture:** Modular Monolith với layered architecture (Controller → Service → Repository)

---

## 🧭 Navigation

### 🟢 Phase 1: Core System (MVP) - TRỌNG TÂM
| File | Description | Priority | Skills |
|------|-------------|----------|--------|
| [01-architecture.md](01-architecture.md) | Layered architecture & Modular monolith | P0 | `backend-architect` |
| [02-authentication.md](02-authentication.md) | JWT, bcrypt & Security basics | P0 | `backend-security-coder` |
| [03-multi-tenant.md](03-multi-tenant.md) | Organization isolation (Multi-tenant) | P0 | `backend-architect` |
| [04-projects-module.md](04-projects-module.md) | Projects (Reference CRUD) | P0 | `backend-architect` |
| [05-issues-module.md](05-issues-module.md) | Issues/Bugs management | P1 | `nodejs-best-practices` |
| [06-tasks-module.md](06-tasks-module.md) | Task management & Assignments | P1 | `nodejs-best-practices` |
| [15-organizations-module.md](15-organizations-module.md) | **NEW:** Organizations CRUD | P0 | `backend-architect` |
| [16-members-module.md](16-members-module.md) | **NEW:** Membership & Roles | P0 | `backend-architect` |
| [17-master-data-module.md](17-master-data-module.md) | **NEW:** Dynamic Data (Tags, Cats) | P1 | `nodejs-best-practices` |
| [11-security-checklist.md](11-security-checklist.md) | Security audit for Core | P0 | `backend-security-coder` |

### 🚀 Phase 2: AI & Scale-up (Future Expansion)
| File | Description | Note | Skills |
|------|-------------|------|--------|
| [07-documents-module.md](07-documents-module.md) | Knowledge Base & RAG Prep | Scale-up | `backend-architect` |
| [08-agents-module.md](08-agents-module.md) | AI Agent Personas | Scale-up | `nodejs-best-practices` |
| [09-chat-module.md](09-chat-module.md) | AI Chat Interface | Scale-up | `backend-architect` |
| [10-workflows-module.md](10-workflows-module.md) | Automation & State Machine | Scale-up | `backend-architect` |
| [12-rag-service.md](12-rag-service.md) | **NEW:** Knowledge Engine | Scale-up | `backend-architect` |
| [13-ai-gateway.md](13-ai-gateway.md) | **NEW:** AI Orchestration | Scale-up | `backend-architect` |
| [14-ai-tools-specification.md](14-ai-tools-specification.md) | **NEW:** AI Tools Specification | Scale-up | `backend-architect` |
| [18-api-routes-map.md](18-api-routes-map.md) | **NEW:** Full API Roadmap | P0 | `backend-architect` |

---

## 🎯 Quick Start

### For New Developers

1. **Understand Architecture:** Start with [01-architecture.md](01-architecture.md)
2. **Setup Security:** Read [02-authentication.md](02-authentication.md) for JWT setup
3. **Learn Patterns:** Study [04-projects-module.md](04-projects-module.md) as reference
4. **Build Features:** Apply same pattern to other modules

### For Security Review

1. [02-authentication.md](02-authentication.md) - Auth implementation
2. [03-multi-tenant.md](03-multi-tenant.md) - Tenant isolation
3. [11-security-checklist.md](11-security-checklist.md) - Security audit

### For Architecture Review

1. [01-architecture.md](01-architecture.md) - Layered architecture
2. [04-projects-module.md](04-projects-module.md) - Reference implementation
3. [10-workflows-module.md](10-workflows-module.md) - Complex patterns

---

## 📐 Module Pattern (All Features Follow This)

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
│   ├── create-{feature}.dto.ts
│   └── update-{feature}.dto.ts
├── routes/
│   └── {feature}.routes.ts
└── index.ts
```

**Flow:** `Request → Controller → Service → Repository → Database`

---

## 🔧 Skills Usage

### Creating New Module

```
@backend-architect @nodejs-best-practices
Create {feature} module following project pattern:
- Controller layer with validation
- Service layer with business logic
- Repository layer with tenant isolation
- DTOs with Zod schemas
```

### Security Audit

```
@backend-security-coder
Audit {feature} module:
- SQL injection prevention
- XSS protection
- CSRF tokens
- Rate limiting
- Input validation
```

---

## 📚 Related Documents

- [../../basics/step-3-architecture.md](../../basics/step-3-architecture.md) - High-level architecture
- [../../basics/step-5-data-design.md](../../basics/step-5-data-design.md) - Database schema + API contracts
- [../common/v1-setup-and-workflows.md](../common/v1-setup-and-workflows.md) - Project setup
- [../database/v1-database-security.md](../database/v1-database-security.md) - Database security

---

## 📝 Notes

- **All modules** MUST implement tenant isolation (`organization_id` filter)
- **All endpoints** MUST validate input với Zod schemas
- **All services** MUST log important actions
- **All repositories** MUST check tenant ownership before update/delete

---

*Last Updated: 2026-02-11*
