# Step 7: Implementation Plan

**Date:** 2026-02-11  
**Status:** ✅ Ready for Implementation

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

## Phase 1: Project Setup & Core Infrastructure

**Duration:** 1-2 tuần  
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

## Phase 2: Authentication & Authorization

**Duration:** 1 tuần  
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

## Phase 3: Multi-Tenant (Organizations & Members)

**Duration:** 1 tuần  
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

## Phase 4: Project Management Core

**Duration:** 2 tuần  
**Goal:** Projects, Issues, Tasks, Comments

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

## Phase 5: Documents & Knowledge Base

**Duration:** 1-2 tuần  
**Goal:** Document management + vector embeddings

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

## Phase 6: AI Agents & Chat

**Duration:** 2-3 tuần  
**Goal:** AI Agents + RAG-based Chat

### Tasks

#### 6.1 Database Migrations
```bash
[ ] Create agents migration
[ ] Create chats migration
[ ] Create messages migration
```

#### 6.2 AI Agents Module
```bash
[ ] agent.model.ts
[ ] agent.controller.ts
[ ] agent.service.ts
[ ] llm.service.ts (OpenAI/Claude client)
[ ] System prompt templates (PM, Dev, Reviewer)
```

**System Prompts:**
```typescript
// apps/api/src/modules/agents/prompts/pm.prompt.ts
export const PM_SYSTEM_PROMPT = `
You are a Project Manager AI assistant for WorkflowHub.
Your role: Help manage projects, break down issues into tasks, track progress.
Capabilities: Create tasks, update statuses, analyze project health.
Context: You have access to project documents, issues, and tasks.
`;

// Similar for Developer, Reviewer roles
```

#### 6.3 Chat Module with RAG
```bash
[ ] chat.controller.ts
[ ] chat.service.ts
[ ] ai-orchestrator.service.ts (RAG pipeline)
[ ] vector-search.service.ts
```

**RAG Pipeline:**
```typescript
export class AIOrchestrator {
  async processQuery(
    chatId: string,
    query: string,
    context: { organizationId: string; projectId?: string }
  ): Promise<string> {
    // 1. Generate query embedding
    const queryEmbedding = await this.embedQuery(query);
    
    // 2. Retrieve relevant documents from Chroma
    const results = await this.chromaCollection.query({
      queryEmbeddings: [queryEmbedding],
      nResults: 5,
      where: {
        organization_id: context.organizationId,
        ...(context.projectId && { project_id: context.projectId }),
      },
    });
    
    // 3. Build context prompt
    const contextPrompt = this.buildContextPrompt(results);
    
    // 4. Call LLM
    const response = await this.llmService.chat({
      messages: [
        { role: 'system', content: SYSTEM_PROMPT + contextPrompt },
        { role: 'user', content: query },
      ],
    });
    
    // 5. Save message with sources
    await this.saveMessage(chatId, query, response, results.ids);
    
    return response;
  }
}
```

#### 6.4 Frontend Chat Components
```bash
[ ] Chat sidebar (chat list)
[ ] Chat window (message thread)
[ ] Message input (with loading state)
[ ] Source citations display
[ ] Agent selector
```

### Verification

- [ ] Agents can be created with custom prompts
- [ ] Chat creates new conversation
- [ ] POST /api/chats/:id/messages triggers RAG
- [ ] Chroma returns relevant documents
- [ ] LLM responds with context
- [ ] Message sources stored (document IDs)
- [ ] Frontend: Chat UI sends/receives messages
- [ ] Frontend: Sources displayed with responses
- [ ] AI responses accurate (based on knowledge base)

---

## Phase 7: Workflow Engine

**Duration:** 2-3 tuần  
**Goal:** Workflow automation

### Tasks

#### 7.1 Database Migrations
```bash
[ ] Create workflow_templates migration
[ ] Create workflow_instances migration
[ ] Create workflow_step_logs migration
```

#### 7.2 Workflow Module
```bash
[ ] workflow-template.model.ts
[ ] workflow-instance.model.ts
[ ] workflow.controller.ts
[ ] workflow.service.ts
[ ] rule-engine.service.ts
[ ] action-executor.service.ts
```

#### 7.3 Workflow Engine Logic
```typescript
export class WorkflowService {
  async executeWorkflow(instanceId: string): Promise<void> {
    const instance = await WorkflowInstance.findByPk(instanceId, {
      include: [WorkflowTemplate],
    });
    
    const { steps } = instance.template;
    
    for (let i = 0; i < steps.length; i++) {
      const step = steps[i];
      
      await this.logStepStart(instance.id, i, step.name);
      
      try {
        // Execute step based on type
        const result = await this.executeStep(step, instance.input_data);
        
        await this.logStepComplete(instance.id, i, result);
        
        // Check onSuccess/onFailure
        if (step.onSuccess) {
          i = step.onSuccess - 1; // Jump to next step
        }
      } catch (error) {
        await this.logStepFailed(instance.id, i, error);
        
        if (step.onFailure) {
          i = step.onFailure - 1;
        } else {
          throw error; // Stop workflow
        }
      }
    }
    
    await instance.update({ status: 'completed' });
  }
}
```

#### 7.4 Workflow Triggers
```bash
[ ] Manual trigger (button click)
[ ] Event-based trigger (issue created, task completed)
[ ] Scheduled trigger (cron jobs - optional)
[ ] AI-triggered (agent completes action - optional)
```

#### 7.5 Frontend Workflow Builder
```bash
[ ] Workflow template designer (drag-drop - optional)
[ ] Workflow list
[ ] Workflow instance status viewer
[ ] Manual trigger button
```

### Verification

- [ ] Workflow templates can be created
- [ ] POST /api/workflows/:id/trigger starts instance
- [ ] Workflow executes steps in order
- [ ] Step results logged
- [ ] Error handling works (onFailure)
- [ ] Event triggers work (e.g., issue created → workflow)
- [ ] Frontend: Workflow list displays
- [ ] Frontend: Manual trigger works
- [ ] Frontend: Instance status updates

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
Phase 1: Infrastructure
├── Monorepo setup
├── Database connection
└── Docker environment

Phase 2: Authentication
└── JWT auth system

Phase 3: Multi-tenant
├── Organizations
└── Members

Phase 4: Core Project Management
├── Projects
├── Issues
├── Tasks
└── Comments

Phase 5: Knowledge Base
├── Documents
└── Vector embeddings (Chroma)

Phase 6: AI Intelligence
├── AI Agents
└── Chat with RAG

Phase 7: Automation
└── Workflow Engine

Phase 8: Production Ready
├── Dashboard
├── Tests
├── Security
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

**Minimum MVP để test end-to-end:**
1. Auth (login)
2. Organizations (create org)
3. Projects (create project)
4. Documents (upload docs)
5. Embeddings (process docs)
6. Chat with AI (query docs)

**Total time:** ~4-6 tuần cho critical path MVP

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

*Document Version: 1.0*  
*Last Updated: 2026-02-13*
