# Step 4: Directory Structure Definition

**Date:** 2026-02-07  
**Status:** ✅ Completed

## Sources

- Previous: [Step 3 - High-Level Architecture](step-3-high-level-architecture.md)
- Stack: Next.js (Frontend) + Node.js (Backend) + MySQL + Vector DB

---

## 1. Project Root Structure

```
workflowhub/
├── apps/
│   ├── web/                    # Frontend: Next.js
│   └── api/                    # Backend: Node.js
├── packages/
│   └── shared/                 # Shared types, utils
├── docs/                       # Documentation
├── scripts/                    # Build, deploy scripts
├── docker/                     # Docker configs
├── .github/                    # CI/CD workflows
├── package.json                # Monorepo root
├── pnpm-workspace.yaml         # pnpm workspace config
└── README.md
```

---

## 2. Frontend Structure (Next.js)

```
apps/web/
├── src/
│   ├── app/                    # Next.js App Router
│   │   ├── (auth)/             # Auth group layout
│   │   │   ├── login/
│   │   │   └── register/
│   │   ├── (dashboard)/        # Dashboard group layout
│   │   │   ├── projects/
│   │   │   ├── issues/
│   │   │   ├── tasks/
│   │   │   ├── documents/
│   │   │   ├── agents/
│   │   │   ├── workflows/
│   │   │   └── chat/
│   │   ├── layout.tsx
│   │   └── page.tsx
│   │
│   ├── components/             # React Components
│   │   ├── ui/                 # Presentational (Button, Input, Modal)
│   │   ├── features/           # Feature-specific components
│   │   │   ├── projects/
│   │   │   ├── issues/
│   │   │   ├── tasks/
│   │   │   ├── documents/
│   │   │   ├── agents/
│   │   │   ├── workflows/
│   │   │   └── chat/
│   │   └── layouts/            # Layout components
│   │
│   ├── hooks/                  # Custom React hooks
│   │   ├── useAuth.ts
│   │   ├── useTenant.ts
│   │   └── useChat.ts
│   │
│   ├── services/               # API client services
│   │   ├── api.ts              # Base API client
│   │   ├── auth.service.ts
│   │   ├── projects.service.ts
│   │   ├── issues.service.ts
│   │   ├── tasks.service.ts
│   │   ├── documents.service.ts
│   │   ├── agents.service.ts
│   │   ├── workflows.service.ts
│   │   └── chat.service.ts
│   │
│   ├── stores/                 # State management (Zustand/Redux)
│   │   ├── auth.store.ts
│   │   ├── tenant.store.ts
│   │   └── ui.store.ts
│   │
│   ├── types/                  # TypeScript types
│   │   └── index.ts
│   │
│   ├── utils/                  # Utility functions
│   │   └── index.ts
│   │
│   └── constants/              # Constants & configs
│       └── index.ts
│
├── public/                     # Static assets
├── next.config.js
├── tailwind.config.js
├── tsconfig.json
└── package.json
```

### Frontend Architecture Rules

| Rule | Description |
|------|-------------|
| **Page/Container/Presentational** | Pages call services → Container logic → Presentational UI |
| **JSDoc Mandatory** | Every function/component must have JSDoc |
| **No Magic Values** | Constants in `constants/` folder |
| **Services = API Only** | Components không gọi fetch trực tiếp |

---

## 3. Backend Structure (Node.js Modular Monolith)

```
apps/api/
├── src/
│   ├── app.ts                  # Composition root
│   ├── server.ts               # Server entry point
│   │
│   ├── modules/                # Feature modules (Modular Monolith)
│   │   │
│   │   ├── auth/               # Authentication & Authorization
│   │   │   ├── controllers/
│   │   │   │   └── auth.controller.ts
│   │   │   ├── services/
│   │   │   │   └── auth.service.ts
│   │   │   ├── repositories/
│   │   │   │   └── user.repository.ts
│   │   │   ├── dtos/
│   │   │   │   ├── login.dto.ts
│   │   │   │   └── register.dto.ts
│   │   │   ├── middlewares/
│   │   │   │   └── auth.middleware.ts
│   │   │   ├── routes/
│   │   │   │   └── auth.routes.ts
│   │   │   └── index.ts        # Module exports
│   │   │
│   │   ├── tenant/             # Multi-tenant management
│   │   │   ├── controllers/
│   │   │   ├── services/
│   │   │   ├── repositories/
│   │   │   ├── middlewares/
│   │   │   │   └── tenant.middleware.ts
│   │   │   ├── routes/
│   │   │   └── index.ts
│   │   │
│   │   ├── projects/           # Project management
│   │   │   ├── controllers/
│   │   │   ├── services/
│   │   │   ├── repositories/
│   │   │   ├── dtos/
│   │   │   ├── routes/
│   │   │   └── index.ts
│   │   │
│   │   ├── issues/             # Issues (GitHub style)
│   │   │   ├── controllers/
│   │   │   ├── services/
│   │   │   ├── repositories/
│   │   │   ├── dtos/
│   │   │   ├── routes/
│   │   │   └── index.ts
│   │   │
│   │   ├── tasks/              # Task management
│   │   │   ├── controllers/
│   │   │   ├── services/
│   │   │   ├── repositories/
│   │   │   ├── dtos/
│   │   │   ├── routes/
│   │   │   └── index.ts
│   │   │
│   │   ├── documents/          # Documents & Knowledge Base
│   │   │   ├── controllers/
│   │   │   ├── services/
│   │   │   ├── repositories/
│   │   │   ├── dtos/
│   │   │   ├── routes/
│   │   │   └── index.ts
│   │   │
│   │   ├── agents/             # AI Agents
│   │   │   ├── controllers/
│   │   │   ├── services/
│   │   │   │   ├── agent.service.ts
│   │   │   │   └── llm.service.ts
│   │   │   ├── repositories/
│   │   │   ├── dtos/
│   │   │   ├── prompts/        # System prompts templates
│   │   │   ├── routes/
│   │   │   └── index.ts
│   │   │
│   │   ├── workflows/          # Workflow Engine
│   │   │   ├── controllers/
│   │   │   ├── services/
│   │   │   │   ├── workflow.service.ts
│   │   │   │   ├── rule-engine.service.ts
│   │   │   │   └── action-executor.service.ts
│   │   │   ├── repositories/
│   │   │   ├── dtos/
│   │   │   ├── routes/
│   │   │   └── index.ts
│   │   │
│   │   ├── chat/               # Chat with AI
│   │   │   ├── controllers/
│   │   │   ├── services/
│   │   │   │   ├── chat.service.ts
│   │   │   │   └── ai-orchestrator.service.ts
│   │   │   ├── repositories/
│   │   │   ├── dtos/
│   │   │   ├── routes/
│   │   │   └── index.ts
│   │   │
│   │   └── vector-store/       # RAG / Vector DB
│   │       ├── services/
│   │       │   └── vector.service.ts
│   │       ├── repositories/
│   │       │   └── embedding.repository.ts
│   │       └── index.ts
│   │
│   ├── shared/                 # Shared utilities
│   │   ├── database/
│   │   │   ├── connection.ts
│   │   │   ├── migrations/
│   │   │   └── seeds/
│   │   ├── middleware/
│   │   │   ├── error-handler.middleware.ts
│   │   │   ├── validation.middleware.ts
│   │   │   └── rate-limiter.middleware.ts
│   │   ├── utils/
│   │   │   ├── logger.ts
│   │   │   └── crypto.ts
│   │   ├── types/
│   │   │   └── index.ts
│   │   ├── constants/
│   │   │   └── index.ts
│   │   └── errors/
│   │       └── http-errors.ts
│   │
│   └── config/                 # App configuration
│       ├── index.ts
│       ├── database.config.ts
│       └── ai.config.ts
│
├── tests/                      # Tests
│   ├── unit/
│   ├── integration/
│   └── e2e/
│
├── .env.example
├── tsconfig.json
├── jest.config.js
└── package.json
```

### Backend Module Pattern

```
module/
├── controllers/    # HTTP request handlers
├── services/       # Business logic
├── repositories/   # Data access
├── dtos/           # Data Transfer Objects
├── middlewares/    # Module-specific middleware
├── routes/         # Route definitions
└── index.ts        # Module exports
```

### Backend Architecture Rules

| Rule | Description |
|------|-------------|
| **Controller → Service → Repository** | Strict layer separation |
| **Module Independence** | Modules không import trực tiếp từ nhau |
| **DTO Validation** | Tất cả input phải qua DTO validation |
| **Tenant Context** | Mọi request phải có tenant context |
| **JSDoc + Error Handling** | Mandatory cho tất cả functions |

---

## 4. Shared Package

```
packages/shared/
├── src/
│   ├── types/                  # Shared TypeScript types
│   │   ├── entities/
│   │   │   ├── user.type.ts
│   │   │   ├── organization.type.ts
│   │   │   ├── project.type.ts
│   │   │   ├── issue.type.ts
│   │   │   ├── task.type.ts
│   │   │   ├── document.type.ts
│   │   │   ├── agent.type.ts
│   │   │   └── workflow.type.ts
│   │   ├── api/
│   │   │   ├── request.type.ts
│   │   │   └── response.type.ts
│   │   └── index.ts
│   │
│   ├── constants/              # Shared constants
│   │   ├── roles.constant.ts
│   │   ├── status.constant.ts
│   │   └── index.ts
│   │
│   └── utils/                  # Shared utilities
│       └── index.ts
│
├── tsconfig.json
└── package.json
```

---

## 5. Key Decisions Summary

| Decision | Choice | Reason |
|----------|--------|--------|
| **Monorepo** | pnpm workspace | Shared types, unified versioning |
| **Module Pattern** | Feature-based | Aligned với Modular Monolith |
| **Frontend State** | Zustand/Redux | Lightweight, predictable |
| **Backend ORM** | Prisma/TypeORM | Type-safe, migrations |
| **API Style** | REST (có thể thêm GraphQL sau) | Simple, well-known |

---

## 6. Related Documents

- [Step 3 - High-Level Architecture](step-3-high-level-architecture.md)
- [Step 3c - Boundaries](step-3c-boundaries.md)
- [Step 2 - MVP Scope](step-2-mvp-scope.md)

---

## 7. GitHub Issues

- #1: Tạo repo và lập kế hoạch
- #2: Step 2 - MVP Scope Clarification
- #3: Step 3 - High-Level Architecture
- #4: Step 3b - Data Flows
- #5: Step 3c - Boundaries
- #6: Step 3d - Architecture Summary
- #7: Step 4 - Directory Structure Definition

---

*Document Version: 1.0*
*Last Updated: 2026-02-07*
