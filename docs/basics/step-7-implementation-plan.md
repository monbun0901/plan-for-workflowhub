# Step 7: Implementation Plan

**Date:** 2026-02-11  
**Updated:** 2026-02-13  
**Status:** 🔄 Implementation In Progress (Phase 7 Complete, Frontend Pending)

> **📌 Note:** Đây là kế hoạch triển khai **HIGH-LEVEL** gồm 8 phases.  
> Chi tiết xem tại [../details/](../details/).


## Tài liệu liên quan

- Trước đó: [Step 6 - Technical Checkpoint](step-6-technical-checkpoint.md)
- Database & API: [Step 5 - Data Design](step-5-data-design.md)
- Architecture: [Step 4 - Directory Structure](step-4-directory-structure.md)
- Tech Stack: [Final Tech Stack](../final-tech-stack.md)

---

## Overview

Kế hoạch triển khai WorkflowHub theo 7 phases, từ setup project đến deployment production-ready.

**Timeline:** Flexible (no hard deadline)  
**Approach:** Incremental, module-by-module  
**Testing:** Continuous (unit + integration tests mỗi phase)

---

## Phase 1: Project Setup & Core Infrastructure ✅ COMPLETED

**Duration:** 1-2 tuần → Actual: 1 tuần  
**Goal:** Setup monorepo, database, và core utilities

### Tasks

#### 1.1 Monorepo Setup
```bash
# Initialize pnpm workspace
[ ] Create workflowhub/ root
[ ] Initialize pnpm workspace (pnpm-workspace.yaml)
[ ] Setup apps/web (Next.js)
[ ] Setup apps/api (Express.js)
[ ] Setup packages/shared (shared types)
[ ] Configure TypeScript (tsconfig.json for each package)
[ ] Setup ESLint + Prettier
[ ] Initialize Git repository
```

**Files to create:**
- `pnpm-workspace.yaml`
- `apps/web/package.json`
- `apps/api/package.json`
- `packages/shared/package.json`
- `.gitignore`
- `README.md`

#### 1.2 Database Setup
```bash
# MySQL + Sequelize
[ ] Install Sequelize CLI
[ ] Configure Sequelize (.sequelizerc, config/database.js)
[ ] Create MySQL database (workflowhub_dev)
[ ] Setup connection (apps/api/src/shared/database/connection.ts)
[ ] Test database connection
```

**Dependencies:**
```json
{
  "sequelize": "^6.35.0",
  "sequelize-cli": "^6.6.0",
  "mysql2": "^3.6.0"
}
```

#### 1.3 Redis Setup (Optional for Phase 1)
```bash
[ ] Install Redis client
[ ] Configure Redis connection
[ ] Test caching layer
```

#### 1.4 Docker Setup
```bash
[ ] Create Dockerfile (apps/api)
[ ] Create Dockerfile (apps/web)
[ ] Create docker-compose.yml (MySQL, Redis, API, Web)
[ ] Test Docker containers startup
```

**docker-compose.yml:**
```yaml
services:
  mysql:
    image: mysql:8
    environment:
      MYSQL_DATABASE: workflowhub_dev
      MYSQL_ROOT_PASSWORD: secret
    ports:
      - "3306:3306"
  
  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
  
  api:
    build: ./apps/api
    ports:
      - "5000:5000"
    depends_on:
      - mysql
      - redis
  
  web:
    build: ./apps/web
    ports:
      - "3000:3000"
    depends_on:
      - api
```

#### 1.5 Shared Utilities
```bash
[ ] Logger utility (Winston)
[ ] Error handler middleware
[ ] Validation middleware (Zod)
[ ] Response formatter
[ ] Constants files
```

**Files to create:**
- `apps/api/src/shared/utils/logger.ts`
- `apps/api/src/shared/middleware/error-handler.middleware.ts`
- `apps/api/src/shared/middleware/validation.middleware.ts`
- `apps/api/src/shared/utils/response.ts`
- `apps/api/src/shared/constants/index.ts`

### Verification

- [ ] `pnpm install` runs successfully
- [ ] `docker-compose up` starts all services
- [ ] Database connection successful
- [ ] Logger writes to console and file
- [ ] Error middleware catches and formats errors

---

## Phase 2: Authentication & Authorization ✅ COMPLETED

**Duration:** 1 tuần → Actual: 3 ngày  
**Goal:** User authentication với JWT

### Tasks

#### 2.1 User Model & Migration
```bash
[ ] Create users migration (Sequelize CLI)
[ ] Create User model (apps/api/src/modules/auth/models/user.model.ts)
[ ] Run migration: npx sequelize-cli db:migrate
```

**Migration:**
```sql
CREATE TABLE users (
  id VARCHAR(36) PRIMARY KEY,
  email VARCHAR(255) UNIQUE NOT NULL,
  password_hash VARCHAR(255) NOT NULL,
  display_name VARCHAR(100) NOT NULL,
  avatar_url VARCHAR(500),
  status ENUM('active', 'inactive', 'suspended') DEFAULT 'active',
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);
```

#### 2.2 Auth Module Implementation
```bash
# Controllers
[ ] auth.controller.ts (register, login, logout, refresh, me)

# Services
[ ] auth.service.ts (business logic)
[ ] jwt.service.ts (token generation/verification)
[ ] password.service.ts (bcrypt hashing)

# DTOs (Zod schemas)
[ ] register.dto.ts
[ ] login.dto.ts

# Middleware
[ ] auth.middleware.ts (JWT verification)

# Routes
[ ] auth.routes.ts
```

**Key Files:**
- `apps/api/src/modules/auth/controllers/auth.controller.ts`
- `apps/api/src/modules/auth/services/auth.service.ts`
- `apps/api/src/modules/auth/services/jwt.service.ts`
- `apps/api/src/modules/auth/dtos/register.dto.ts`
- `apps/api/src/modules/auth/middlewares/auth.middleware.ts`

#### 2.3 JWT Configuration
```typescript
// apps/api/src/config/jwt.config.ts
export const jwtConfig = {
  accessTokenSecret: process.env.JWT_ACCESS_SECRET,
  refreshTokenSecret: process.env.JWT_REFRESH_SECRET,
  accessTokenExpiry: '15m',
  refreshTokenExpiry: '7d',
};
```

#### 2.4 Frontend Auth Components
```bash
# Pages
[ ] apps/web/src/app/(auth)/login/page.tsx
[ ] apps/web/src/app/(auth)/register/page.tsx

# Hooks
[ ] apps/web/src/hooks/useAuth.ts

# Store
[ ] apps/web/src/stores/auth.store.ts (Zustand)

# Service
[ ] apps/web/src/services/auth.service.ts (Axios API calls)
```

### Verification

- [ ] POST /api/auth/register creates user
- [ ] POST /api/auth/login returns JWT tokens
- [ ] GET /api/auth/me returns user info (with valid JWT)
- [ ] Middleware rejects invalid/expired tokens
- [ ] Frontend: Login/Register forms work
- [ ] Frontend: Auth state persists (Zustand)

---

## Phase 3: Multi-Tenant (Organizations & Members) ✅ COMPLETED

**Duration:** 1 tuần → Actual: 3 ngày  
**Goal:** Multi-tenant isolation

### Tasks

#### 3.1 Database Migrations
```bash
[ ] Create organizations migration
[ ] Create members migration
[ ] Create roles migration (if needed)
[ ] Run migrations
```

#### 3.2 Tenant Module Implementation
```bash
# Models
[ ] organization.model.ts
[ ] member.model.ts

# Controllers
[ ] organization.controller.ts
[ ] member.controller.ts

# Services
[ ] organization.service.ts
[ ] member.service.ts
[ ] invitation.service.ts

# Middleware
[ ] tenant.middleware.ts (extract/validate organization_id)

# DTOs
[ ] create-organization.dto.ts
[ ] invite-member.dto.ts

# Routes
[ ] organization.routes.ts
[ ] member.routes.ts
```

#### 3.3 Tenant Isolation Strategy
```typescript
// Middleware: Inject tenant context
export const tenantMiddleware = async (req, res, next) => {
  const { orgId } = req.params;
  const userId = req.user.id;
  
  // Verify user belongs to organization
  const member = await Member.findOne({
    where: { user_id: userId, organization_id: orgId }
  });
  
  if (!member) {
    throw new ForbiddenError('Not authorized for this organization');
  }
  
  req.tenant = { organizationId: orgId, role: member.role };
  next();
};
```

#### 3.4 Frontend Organization Components
```bash
[ ] Organization selector (dropdown)
[ ] Organization settings page
[ ] Member management page
[ ] Invite member modal
```

### Verification

- [ ] POST /api/organizations creates organization
- [ ] GET /api/organizations lists user's organizations
- [ ] POST /api/organizations/:orgId/members invites member
- [ ] Tenant middleware blocks cross-tenant access
- [ ] Frontend: Organization switcher works
- [ ] Frontend: Member list displays correctly

---

## Phase 4: Project Management Core ✅ COMPLETED

**Duration:** 2 tuần → Actual: 1 tuần  
**Goal:** Projects, Issues, Workflow Statuses, Comments, Labels, Documents

### Tasks

#### 4.1 Database Migrations
```bash
[ ] Create projects migration
[ ] Create issues migration
[ ] Create tasks migration
[ ] Create comments migration
```

#### 4.2 Modules Implementation

**Projects Module:**
```bash
[ ] project.model.ts
[ ] project.controller.ts
[ ] project.service.ts
[ ] project.routes.ts
```

**Issues Module:**
```bash
[ ] issue.model.ts
[ ] issue.controller.ts
[ ] issue.service.ts
[ ] issue.routes.ts
[ ] Auto-increment issue number per project
```

**Tasks Module:**
```bash
[ ] task.model.ts
[ ] task.controller.ts
[ ] task.service.ts
[ ] task.routes.ts
```

**Comments Module:**
```bash
[ ] comment.model.ts (polymorphic: issue/task)
[ ] comment.controller.ts
[ ] comment.service.ts
```

#### 4.3 Frontend Pages
```bash
# Projects
[ ] /projects - List projects
[ ] /projects/[id] - Project detail
[ ] /projects/[id]/settings

# Issues
[ ] /projects/[id]/issues - List issues
[ ] /projects/[id]/issues/[number] - Issue detail
[ ] Issue create/edit form

# Tasks
[ ] /projects/[id]/tasks - Kanban board
[ ] Task detail modal
[ ] Task create/edit form
```

### Verification

- [ ] CRUD operations work for Projects
- [ ] CRUD operations work for Issues
- [ ] Issue numbering auto-increments (#1, #2, ...)
- [ ] CRUD operations work for Tasks
- [ ] Comments can be added to issues/tasks
- [ ] Tenant isolation verified (organization_id filter)
- [ ] Frontend: Project list displays
- [ ] Frontend: Issue board works
- [ ] Frontend: Task kanban board works

---

## Phase 5: Collaboration & Knowledge Base ✅ COMPLETED

**Duration:** 1-2 tuần → Actual: 3 ngày  
**Goal:** Comments, Labels, Documents (Vector embeddings deferred to Phase 8)

### Tasks

#### 5.1 Database Migrations
```bash
[ ] Create documents migration
[ ] Create document_versions migration
```

#### 5.2 Documents Module
```bash
[ ] document.model.ts
[ ] document.controller.ts
[ ] document.service.ts
[ ] document.routes.ts
[ ] Markdown rendering (frontend)
[ ] Document versioning logic
```

#### 5.3 Vector Store Setup (Chroma)
```bash
[ ] Install chromadb client
[ ] Configure Chroma connection
[ ] Create collection: workflowhub_docs
[ ] Setup embedding service
```

**Dependencies:**
```json
{
  "chromadb": "^0.5.0",
  "openai": "^4.20.0"  // Fallback embeddings (MVP)
}
```

#### 5.4 Embedding Pipeline
```bash
[ ] embedding.service.ts
    - Chunk documents (500-1000 tokens)
    - Generate embeddings (OpenAI text-embedding-3)
    - Store in Chroma with metadata
    
[ ] embedding.job.ts (BullMQ queue - optional)
    - Background processing
    
[ ] Trigger on document create/update
```

**Embedding Logic:**
```typescript
export class EmbeddingService {
  async embedDocument(documentId: string): Promise<void> {
    const doc = await Document.findByPk(documentId);
    
    // Chunk content
    const chunks = this.chunkText(doc.content, 800);
    
    // Generate embeddings
    const embeddings = await this.generateEmbeddings(chunks);
    
    // Store in Chroma
    await this.chromaCollection.add({
      ids: chunks.map((_, i) => `${documentId}_${i}`),
      embeddings: embeddings,
      metadatas: chunks.map((chunk, i) => ({
        organization_id: doc.organization_id,
        project_id: doc.project_id,
        document_id: documentId,
        chunk_index: i,
        source_type: 'document',
      })),
      documents: chunks,
    });
    
    await doc.update({ embedding_status: 'completed' });
  }
}
```

#### 5.5 Frontend Document Components
```bash
[ ] Document editor (Markdown)
[ ] Document viewer (rendered Markdown)
[ ] Document tree navigation
[ ] Version history viewer
```

### Verification

- [ ] Documents can be created/edited
- [ ] Markdown renders correctly
- [ ] Document versions saved
- [ ] Embeddings generated on document save
- [ ] Chroma stores embeddings with correct metadata
- [ ] Tenant isolation in vector store (organization_id filter)
- [ ] Frontend: Document editor works
- [ ] Frontend: Document tree displays

---

## Phase 6: AI Agents & Chat ✅ COMPLETED

**Duration:** 2-3 tuần → Actual: 2 ngày  
**Goal:** Multi-provider AI Agents + Chat (RAG deferred)

### Actual Implementation

#### 6.1 Database (Migration Batch 4)
- [x] `agents` table (provider, model, system_prompt, type)
- [x] `chats` table (org, project, agent, user scoped)
- [x] `messages` table (role, content, token_count)

#### 6.2 Multi-Provider Architecture
```
LLMProvider (Abstract Interface)
├── OpenAIProvider  (native fetch → OpenAI API)
├── GeminiProvider  (native fetch → Gemini API)
└── ProviderFactory (registry pattern, env-based API keys)
```

#### 6.3 Core Services
- [x] `AgentService` — CRUD + auto system prompt by type
- [x] `ChatService` — Chat lifecycle, message flow, auto-titling
- [x] `AIOrchestrator` — Central AI pipeline (build context → select provider → call LLM)
- [x] System prompts: `general`, `pm`, `developer`, `reviewer`

#### 6.4 API Endpoints
| Method | Endpoint | Description |
|--------|----------|-------------|
| POST   | `/:orgId/agents` | Create agent |
| GET    | `/:orgId/agents` | List agents |
| GET    | `/:orgId/agents/providers` | List available providers |
| POST   | `/:orgId/chats` | Create chat |
| GET    | `/:orgId/chats` | List chats |
| POST   | `/:orgId/chats/:id/messages` | Send message → AI responds |

### Verification ✅
- [x] Agent CRUD works
- [x] Multi-provider (OpenAI + Gemini) tested
- [x] Chat creates conversations
- [x] AI responds via Gemini API
- [x] Message history maintained (capped at 20)
- [ ] RAG pipeline (deferred — requires ChromaDB setup)
- [ ] Frontend Chat UI (Phase 8)

---

## Phase 7: Workflow Engine ✅ COMPLETED

**Duration:** 2-3 tuần → Actual: 1 ngày  
**Goal:** Workflow-centric automation (AI là tool trong pipeline)

### 🏛️ Architecture Decision: Workflow-Centric

```
Event (Issue Created/Updated)
        │
        ▼
   WorkflowTriggerService (EventEmitter)
        │ findMatchingWorkflows()
        ▼
   WorkflowExecutor.executeRun()
        │ loop through steps (context passing)
        ▼
   Action Registry
   ├── update_issue   ✅ — Update status/assignee/priority
   ├── call_ai        ✅ — Call AI Agent (prompt template + issue context)
   ├── create_comment ✅ — Post comment (supports AI output)
   ├── send_email     ⬜ — Phase 8
   └── assign_user    ⬜ — Phase 8
```

**AI Agent = 1 tool trong Workflow pipeline** (không phải trung tâm).
Kết quả step trước chảy vào step sau qua `previousResult`.

### Actual Implementation

#### 7.1 Database (Migration Batch 5)
- [x] `workflows` — Automation rules (trigger_type, scope org/project)
- [x] `workflow_steps` — Actions/conditions (self-ref branching)
- [x] `workflow_runs` — Execution logs (status, duration, JSON logs)

#### 7.2 Models
- [x] `Workflow` — name, trigger_type, is_active, org/project scope
- [x] `WorkflowStep` — type (action/condition/delay), action_type, config JSON, order
- [x] `WorkflowRun` — status (pending→running→success/failed), logs, duration_ms

#### 7.3 Trigger System
- [x] `WorkflowTriggerService` (extends EventEmitter) — Singleton
- [x] Events: `issue_created`, `issue_updated`, `comment_created`
- [x] `IssueService` emit events after create/update

#### 7.4 Executor & Actions
- [x] `WorkflowExecutor` — Step-by-step execution with context passing
- [x] `update_issue` action — Update issue fields from config
- [x] `call_ai` action — Load Agent → interpolate prompt → call AIOrchestrator
- [x] `create_comment` action — Post comment (supports `use_previous_result` from call_ai)

#### 7.5 API Endpoints
| Method | Endpoint | Description |
|--------|----------|-------------|
| POST   | `/:orgId/workflows` | Create workflow with steps |
| GET    | `/:orgId/workflows` | List workflows |
| GET    | `/:orgId/workflows/:id` | Get workflow detail |
| PATCH  | `/:orgId/workflows/:id` | Update workflow |
| DELETE | `/:orgId/workflows/:id` | Delete workflow |

#### 7.6 Example Pipeline
```json
{
  "name": "AI Review on Issue Created",
  "trigger_type": "issue_created",
  "steps": [
    { "order": 1, "action_type": "call_ai", "config": { "agent_id": "xxx", "prompt_template": "Analyze: {{title}}" } },
    { "order": 2, "action_type": "create_comment", "config": { "use_previous_result": true, "author_label": "🤖 PM" } },
    { "order": 3, "action_type": "update_issue", "config": { "priority": "high" } }
  ]
}
```

### Verification ✅
- [x] Workflow CRUD via API (validated with Zod)
- [x] Event triggers work (issue_created → workflow auto-runs)
- [x] Steps execute in order with context passing
- [x] Issue auto-updated by workflow (tested: To Do → Done)
- [x] Execution logs stored in workflow_runs
- [ ] Frontend: Workflow builder UI (Phase 8)

---

## Phase 8: Polish & Deployment

**Duration:** 1-2 tuần  
**Goal:** Production-ready deployment

### Tasks

#### 8.1 Dashboard & Analytics
```bash
[ ] Dashboard page (projects, issues, tasks overview)
[ ] Analytics widgets (charts)
[ ] Activity feed
```

#### 8.2 UI/UX Refinement
```bash
[ ] Responsive design (mobile, tablet)
[ ] Loading states
[ ] Error states
[ ] Empty states
[ ] Toast notifications
```

#### 8.3 Testing
```bash
# Unit Tests
[ ] Auth service tests
[ ] Organization service tests
[ ] RAG pipeline tests (mock LLM)
[ ] Workflow engine tests

# Integration Tests
[ ] API endpoint tests (Supertest)
[ ] Database transaction tests

# E2E Tests (optional)
[ ] Login flow
[ ] Create project flow
[ ] Chat with AI flow
```

#### 8.4 Security Hardening
```bash
[ ] Rate limiting (express-rate-limit)
[ ] CORS configuration
[ ] Helmet.js (security headers)
[ ] Input sanitization
[ ] SQL injection prevention (Sequelize escaping)
```

#### 8.5 Performance Optimization
```bash
[ ] Database query optimization
[ ] N+1 query prevention
[ ] Redis caching (hot data)
[ ] API response compression
```

#### 8.6 Documentation
```bash
[ ] API documentation (Swagger/OpenAPI - optional)
[ ] README.md (setup instructions)
[ ] Architecture diagram
[ ] Deployment guide
```

#### 8.7 CI/CD Pipeline
```bash
# .github/workflows/ci.yml
[ ] Lint check (ESLint)
[ ] Type check (TypeScript)
[ ] Unit tests
[ ] Build check
[ ] Docker image build

# .github/workflows/deploy.yml (optional)
[ ] Deploy to staging
[ ] Deploy to production
```

#### 8.8 Production Deployment
```bash
[ ] Setup production database (MySQL)
[ ] Setup Redis instance
[ ] Setup Chroma vector store
[ ] Configure environment variables
[ ] Deploy API (Railway/Render/VPS)
[ ] Deploy Frontend (Vercel)
[ ] Setup domain & SSL
[ ] Database backup strategy
```

### Verification

- [ ] All tests pass
- [ ] Lighthouse score > 90
- [ ] Security headers present
- [ ] Rate limiting blocks abuse
- [ ] CI/CD pipeline runs successfully
- [ ] Production deployment successful
- [ ] Health check endpoint responds
- [ ] Database backups automated

---

## Module Implementation Order

```
Phase 1: Infrastructure                         ✅ DONE
├── Monorepo setup (pnpm workspace)
├── Database connection (MySQL + Sequelize)
└── Shared utilities (Logger, Error Handler, Validation)

Phase 2: Authentication                          ✅ DONE
└── JWT auth (register, login, refresh, me)

Phase 3: Multi-tenant                            ✅ DONE
├── Organizations (CRUD)
├── Members (invite, roles)
└── Tenant isolation middleware

Phase 4: Project Management Core                 ✅ DONE
├── Projects (CRUD + workflow statuses)
├── Issues (CRUD + filters + ordering)
├── Project Members
└── Workflow Statuses (Kanban columns)

Phase 5: Collaboration & Knowledge Base          ✅ DONE
├── Comments (threaded)
├── Labels (org-scoped, many-to-many)
└── Documents (tree hierarchy, rich text)

Phase 6: AI Agents & Chat                        ✅ DONE
├── Multi-provider (OpenAI + Gemini)
├── AI Orchestrator (centralized LLM)
├── Agent CRUD + system prompts
└── Chat with message history

Phase 7: Workflow Engine (Workflow-Centric)       ✅ DONE
├── Workflows (rules + steps)
├── Trigger system (EventEmitter)
├── Executor (step-by-step + context passing)
└── Actions: update_issue, call_ai, create_comment

Phase 8: Frontend + Polish                       ⬜ NEXT
├── Frontend implementation (React/Next.js)
├── Dashboard & Analytics
├── RAG pipeline (ChromaDB)
├── Testing & Security
└── Deployment
```

---

## Testing Strategy

### Unit Tests (Jest)
```bash
# Target coverage: 80%+
- Services: Business logic
- Utils: Helper functions
- DTOs: Validation schemas
```

### Integration Tests (Supertest)
```bash
# API endpoints
- Auth flow
- CRUD operations
- Tenant isolation
```

### Manual Testing Checklist
```
[ ] Register new user
[ ] Login with correct/incorrect credentials
[ ] Create organization
[ ] Invite member
[ ] Create project
[ ] Create issue
[ ] Create task from issue
[ ] Upload document
[ ] Verify document embedded
[ ] Chat with AI (RAG)
[ ] Verify AI cites sources
[ ] Create workflow
[ ] Trigger workflow manually
[ ] Verify workflow completes
```

---

## Migration Strategy

### Database Migrations (Sequelize)
```bash
# Create migration
npx sequelize-cli migration:generate --name create-users

# Run migrations
npx sequelize-cli db:migrate

# Rollback
npx sequelize-cli db:migrate:undo
```

### Migration Naming Convention
```
YYYYMMDDHHMMSS-descriptive-name.js

Examples:
20260211120000-create-users.js
20260211120100-create-organizations.js
20260211120200-create-projects.js
```

---

## Environment Variables

### Development (.env.development)
```env
# App
NODE_ENV=development
PORT=5000

# Database
DATABASE_URL=mysql://root:secret@localhost:3306/workflowhub_dev

# Redis
REDIS_URL=redis://localhost:6379

# JWT
JWT_ACCESS_SECRET=your-secret-key
JWT_REFRESH_SECRET=your-refresh-secret

# AI (MVP - Cloud)
OPENAI_API_KEY=sk-...
# ANTHROPIC_API_KEY=sk-ant-...  (optional)

# Vector DB
CHROMA_HOST=localhost
CHROMA_PORT=8001
CHROMA_AUTH_TOKEN=whub-chroma-secret-token

# Embedding (Ollama local-first)
OLLAMA_HOST=http://localhost:11434
OLLAMA_EMBED_MODEL=nomic-embed-text

# Frontend
NEXT_PUBLIC_API_URL=http://localhost:5000
```

### Production (.env.production)
```env
NODE_ENV=production
PORT=5000

DATABASE_URL=mysql://user:pass@prod-db:3306/workflowhub_prod
REDIS_URL=redis://prod-redis:6379

JWT_ACCESS_SECRET=<strong-secret>
JWT_REFRESH_SECRET=<strong-refresh-secret>

OPENAI_API_KEY=sk-...

CHROMA_HOST=chromadb
CHROMA_PORT=8001
CHROMA_AUTH_TOKEN=<strong-token>

NEXT_PUBLIC_API_URL=https://api.workflowhub.com
```

---

## Critical Path

**Backend MVP — COMPLETED ✅ (Actual: ~2 tuần)**
1. ✅ Auth (register, login, JWT refresh)
2. ✅ Organizations (create, members, roles)
3. ✅ Projects (CRUD, workflow statuses)
4. ✅ Issues (CRUD, filters, ordering)
5. ✅ Collaboration (comments, labels, documents)
6. ✅ AI Chat (multi-provider, orchestrator)
7. ✅ Workflow Engine (triggers, executor, AI actions)

**Frontend MVP — NEXT:**
1. Layout system + Auth pages
2. Organization & Project UI
3. Issue Board (Kanban)
4. AI Chat interface
5. Workflow builder
6. Dashboard & Analytics

---

## Success Criteria

- [ ] User có thể register/login
- [ ] User có thể tạo organization
- [ ] User có thể tạo project
- [ ] User có thể upload documents
- [ ] Documents được embed vào vector store
- [ ] User có thể chat với AI
- [ ] AI trả lời dựa trên documents (RAG works)
- [ ] Multi-tenant isolation verified (không leak data)
- [ ] Tests pass (unit + integration)
- [ ] Production deployment successful

---

## Tài liệu liên quan

- [Step 6 - Technical Checkpoint](step-6-technical-checkpoint.md)
- [Step 5 - Data Design](step-5-data-design.md)
- [Step 4 - Directory Structure](step-4-directory-structure.md)
- [Final Tech Stack](../final-tech-stack.md)
- [Database Migration Order](../details/database/migration-order.md)
- [Frontend Implementation Steps](../details/frontend/implementation-steps.md)
- [Backend Modules](../details/backend/modules/README.md)
- [Vector DB Plan](../details/backend/services/04-vector-db.md)

---

*Document Version: 2.0*  
*Last Updated: 2026-02-13*  
*Backend Phases 1-7: COMPLETED*  
*Next: Frontend Implementation*
