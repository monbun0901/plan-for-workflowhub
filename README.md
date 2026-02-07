# WorkflowHub

> **Nền tảng quản lý công việc và dự án đa tổ chức tích hợp AI và workflow thông minh**

## Quick Links

- [📋 Project Plan v1](docs/v1-initial-plan.md) - Initial planning document
- [🎯 MVP Scope & Requirements](docs/step-2-mvp-scope.md) - Step 2 clarification
- [🏗️ Architecture - Step 3](docs/step-3-high-level-architecture.md) - Modular Monolith
- [📊 GitHub Issues](https://github.com/monbun0901/plan-for-workflowhub/issues)

## Overview

### Core Features
- ✅ Multi-tenant architecture
- ✅ AI-powered project management
- ✅ Workflow automation
- ✅ RAG-based knowledge base

### Tech Stack
- **Frontend:** Next.js
- **Backend:** Node.js
- **Database:** MySQL

## Documentation Structure

```
docs/
├── v1-initial-plan.md                  # Original project plan
├── step-2-mvp-scope.md                 # Step 2: MVP scope clarification
├── step-3-high-level-architecture.md   # Step 3: Modular Monolith
├── step-3b-data-flows.md               # Data flows (User→AI, AI→Workflow)
├── step-3c-boundaries.md              # Boundaries (Frontend/Backend/AI)
└── step-3d-architecture-summary.md   # Step 3: Complete summary + Golden Rules
```

## 🏆 Golden Rules

1. **AI không bao giờ là root of trust**
2. **Tenant boundary > AI intelligence**
3. **Workflow quyết định bằng rule, không bằng LLM**
4. **Monolith trước, microservices sau**

---

*Last Updated: 2026-02-07*
