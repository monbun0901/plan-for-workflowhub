# Step 5: Data Design (Database Schema + API Contracts)

**Date:** 2026-02-07  
**Status:** ✅ Completed

## Sources

- Previous: [Step 4 - Directory Structure](step-4-directory-structure.md)
- Architecture: [Step 3 - Architecture Design](step-3-architecture.md)

---

## 🎯 Overview

Complete data design for WorkflowHub covering:
- **Part 1:** Entity Relationship Diagram (ERD)
- **Part 2:** Database Schema (MySQL)
- **Part 3:** Vector Store Schema
**Part 4:** REST API Contracts & DTOs

---

## PART 1: Entity Relationship Diagram (ERD)

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                              WORKFLOWHUB ERD                                 │
└─────────────────────────────────────────────────────────────────────────────┘

┌──────────────┐       ┌──────────────┐       ┌──────────────┐
│     USER     │──────▶│   MEMBER     │◀──────│ ORGANIZATION │
└──────────────┘  1:N  └──────────────┘  N:1  └──────────────┘
                              │                      │
                              │                      │ 1:N
                              ▼                      ▼
                       ┌──────────────┐       ┌──────────────┐
                       │    ROLE      │       │   PROJECT    │
                       └──────────────┘       └──────┬───────┘
                                                     │
                    ┌────────────────────────────────┼────────────────────────┐
                    │                                │                        │
                    ▼                                ▼                        ▼
             ┌──────────────┐               ┌──────────────┐          ┌──────────────┐
             │    ISSUE     │──────────────▶│    TASK      │          │  DOCUMENT    │
             └──────────────┘     1:N       └──────────────┘          └──────────────┘
                    │                              │                         │
                    ▼                              ▼                         ▼
             ┌──────────────┐               ┌──────────────┐          ┌──────────────┐
             │   COMMENT    │               │  TASK_LOG    │          │  DOC_VERSION │
             └──────────────┘               └──────────────┘          └──────────────┘


             ┌──────────────┐               ┌──────────────┐          ┌──────────────┐
             │    AGENT     │               │   WORKFLOW   │          │    CHAT      │
             └──────────────┘               └──────────────┘          └──────────────┘
                    │                              │                         │
                    ▼                              ▼                         ▼
             ┌──────────────┐               ┌──────────────┐          ┌──────────────┐
             │ AGENT_ACTION │               │ WF_INSTANCE  │          │   MESSAGE    │
             └──────────────┘               └──────────────┘          └──────────────┘
```

---

## PART 2: Database Schema (MySQL)

### 2.1 User & Auth Tables

#### users

```sql
CREATE TABLE users (
  id            VARCHAR(36) PRIMARY KEY,        -- UUID
  email         VARCHAR(255) UNIQUE NOT NULL,
  password_hash VARCHAR(255) NOT NULL,
  display_name  VARCHAR(100) NOT NULL,
  avatar_url    VARCHAR(500),
  status        ENUM('active', 'inactive', 'suspended') DEFAULT 'active',
  created_at    TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at    TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);
```

#### organizations

```sql
CREATE TABLE organizations (
  id            VARCHAR(36) PRIMARY KEY,
  name          VARCHAR(100) NOT NULL,
  slug          VARCHAR(100) UNIQUE NOT NULL,   -- URL-friendly identifier
  description   TEXT,
  logo_url      VARCHAR(500),
  settings      JSON,                           -- Org-level settings
  created_by    VARCHAR(36) REFERENCES users(id),
  created_at    TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at    TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);
```

#### members

```sql
CREATE TABLE members (
  id              VARCHAR(36) PRIMARY KEY,
  user_id         VARCHAR(36) NOT NULL REFERENCES users(id),
  organization_id VARCHAR(36) NOT NULL REFERENCES organizations(id),
  role            ENUM('owner', 'admin', 'member') DEFAULT 'member',
  status          ENUM('active', 'pending', 'removed') DEFAULT 'pending',
  invited_by      VARCHAR(36) REFERENCES users(id),
  joined_at       TIMESTAMP,
  created_at      TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  
  UNIQUE KEY unique_user_org (user_id, organization_id)
);
```

---

### 2.2 Project Management Tables

#### projects

```sql
CREATE TABLE projects (
  id              VARCHAR(36) PRIMARY KEY,
  organization_id VARCHAR(36) NOT NULL REFERENCES organizations(id),
  name            VARCHAR(100) NOT NULL,
  slug            VARCHAR(100) NOT NULL,
  description     TEXT,
  status          ENUM('active', 'archived', 'deleted') DEFAULT 'active',
  visibility      ENUM('private', 'internal', 'public') DEFAULT 'private',
  settings        JSON,
  created_by      VARCHAR(36) REFERENCES users(id),
  created_at      TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at      TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  
  UNIQUE KEY unique_org_slug (organization_id, slug),
  INDEX idx_org_status (organization_id, status)
);
```

#### issues

```sql
CREATE TABLE issues (
  id              VARCHAR(36) PRIMARY KEY,
  project_id      VARCHAR(36) NOT NULL REFERENCES projects(id),
  organization_id VARCHAR(36) NOT NULL REFERENCES organizations(id),  -- Denormalized for tenant filter
  
  number          INT NOT NULL,                  -- Auto-increment per project (#1, #2, ...)
  title           VARCHAR(200) NOT NULL,
  description     TEXT,
  type            ENUM('bug', 'feature', 'improvement', 'question') DEFAULT 'feature',
  status          ENUM('open', 'in_progress', 'resolved', 'closed') DEFAULT 'open',
  priority        ENUM('low', 'medium', 'high', 'critical') DEFAULT 'medium',
  
  assignee_id     VARCHAR(36) REFERENCES users(id),
  reporter_id     VARCHAR(36) NOT NULL REFERENCES users(id),
  
  labels          JSON,                          -- Array of label strings
  due_date        DATE,
  
  created_at      TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at      TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  closed_at       TIMESTAMP,
  
  UNIQUE KEY unique_project_number (project_id, number),
  INDEX idx_org_status (organization_id, status),
  INDEX idx_project_status (project_id, status)
);
```

#### tasks

```sql
CREATE TABLE tasks (
  id              VARCHAR(36) PRIMARY KEY,
  issue_id        VARCHAR(36) REFERENCES issues(id),
  project_id      VARCHAR(36) NOT NULL REFERENCES projects(id),
  organization_id VARCHAR(36) NOT NULL REFERENCES organizations(id),
  
  title           VARCHAR(200) NOT NULL,
  description     TEXT,
  status          ENUM('todo', 'in_progress', 'review', 'done', 'cancelled') DEFAULT 'todo',
  priority        ENUM('low', 'medium', 'high', 'critical') DEFAULT 'medium',
  
  assignee_id     VARCHAR(36) REFERENCES users(id),
  created_by      VARCHAR(36) NOT NULL REFERENCES users(id),
  
  estimated_hours DECIMAL(5,2),
  actual_hours    DECIMAL(5,2),
  due_date        DATETIME,
  completed_at    TIMESTAMP,
  
  -- AI-generated task tracking
  ai_generated    BOOLEAN DEFAULT FALSE,
  generated_by_agent VARCHAR(36) REFERENCES agents(id),
  
  created_at      TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at      TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  
  INDEX idx_org_status (organization_id, status),
  INDEX idx_issue (issue_id),
  INDEX idx_assignee (assignee_id, status)
);
```

#### documents

```sql
CREATE TABLE documents (
  id              VARCHAR(36) PRIMARY KEY,
  project_id      VARCHAR(36) NOT NULL REFERENCES projects(id),
  organization_id VARCHAR(36) NOT NULL REFERENCES organizations(id),
  
  title           VARCHAR(200) NOT NULL,
  slug            VARCHAR(200) NOT NULL,
  content         LONGTEXT,                      -- Markdown content
  content_type    ENUM('markdown', 'rich_text') DEFAULT 'markdown',
  
  parent_id       VARCHAR(36) REFERENCES documents(id),  -- For nested docs
  order_index     INT DEFAULT 0,
  
  status          ENUM('draft', 'published', 'archived') DEFAULT 'draft',
  version         INT DEFAULT 1,
  
  created_by      VARCHAR(36) NOT NULL REFERENCES users(id),
  updated_by      VARCHAR(36) REFERENCES users(id),
  
  -- Vector embedding status
  embedding_status ENUM('pending', 'processing', 'completed', 'failed') DEFAULT 'pending',
  last_embedded_at TIMESTAMP,
  
  created_at      TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at      TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  
  UNIQUE KEY unique_project_slug (project_id, slug),
  INDEX idx_org (organization_id),
  INDEX idx_embedding (organization_id, embedding_status)
);

CREATE TABLE document_versions (
  id              VARCHAR(36) PRIMARY KEY,
  document_id     VARCHAR(36) NOT NULL REFERENCES documents(id),
  version         INT NOT NULL,
  title           VARCHAR(200) NOT NULL,
  content         LONGTEXT,
  created_by      VARCHAR(36) REFERENCES users(id),
  created_at      TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  
  UNIQUE KEY unique_doc_version (document_id, version)
);
```

---

### 2.3 AI Tables

#### agents

```sql
CREATE TABLE agents (
  id              VARCHAR(36) PRIMARY KEY,
  organization_id VARCHAR(36) NOT NULL REFERENCES organizations(id),
  project_id      VARCHAR(36) REFERENCES projects(id),  -- NULL = org-wide agent
  
  name            VARCHAR(100) NOT NULL,
  role            ENUM('pm', 'developer', 'reviewer', 'analyst', 'custom') NOT NULL,
  description     TEXT,
  
  system_prompt   TEXT NOT NULL,                 -- Base system prompt
  capabilities    JSON,                          -- Allowed actions/tools
  
  status          ENUM('active', 'disabled') DEFAULT 'active',
  
  -- LLM settings
  model           VARCHAR(50) DEFAULT 'gpt-4',
  temperature     DECIMAL(2,1) DEFAULT 0.7,
  max_tokens      INT DEFAULT 4096,
  
  created_by      VARCHAR(36) REFERENCES users(id),
  created_at      TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at      TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  
  INDEX idx_org (organization_id),
  INDEX idx_project (project_id)
);
```

#### chats & messages

```sql
CREATE TABLE chats (
  id              VARCHAR(36) PRIMARY KEY,
  organization_id VARCHAR(36) NOT NULL REFERENCES organizations(id),
  project_id      VARCHAR(36) REFERENCES projects(id),
  
  user_id         VARCHAR(36) NOT NULL REFERENCES users(id),
  agent_id        VARCHAR(36) REFERENCES agents(id),
  
  title           VARCHAR(200),
  status          ENUM('active', 'archived') DEFAULT 'active',
  
  -- Context
  context_type    ENUM('general', 'project', 'issue', 'task', 'document'),
  context_id      VARCHAR(36),                   -- ID of the context entity
  
  created_at      TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at      TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  
  INDEX idx_user (user_id),
  INDEX idx_org (organization_id)
);

CREATE TABLE messages (
  id              VARCHAR(36) PRIMARY KEY,
  chat_id         VARCHAR(36) NOT NULL REFERENCES chats(id),
  organization_id VARCHAR(36) NOT NULL REFERENCES organizations(id),
  
  role            ENUM('user', 'assistant', 'system') NOT NULL,
  content         TEXT NOT NULL,
  
  -- AI metadata
  model_used      VARCHAR(50),
  tokens_input    INT,
  tokens_output   INT,
  
  -- RAG metadata
  sources         JSON,                          -- Retrieved document IDs
  
  created_at      TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  
  INDEX idx_chat (chat_id),
  INDEX idx_org (organization_id)
);
```

#### workflow tables

```sql
CREATE TABLE workflow_templates (
  id              VARCHAR(36) PRIMARY KEY,
  organization_id VARCHAR(36) NOT NULL REFERENCES organizations(id),
  project_id      VARCHAR(36) REFERENCES projects(id),
  
  name            VARCHAR(100) NOT NULL,
  description     TEXT,
  
  trigger_type    ENUM('manual', 'event', 'scheduled', 'ai') NOT NULL,
  trigger_config  JSON,                          -- Trigger conditions
  
  steps           JSON NOT NULL,                 -- Workflow steps definition
  
  status          ENUM('active', 'disabled', 'draft') DEFAULT 'draft',
  version         INT DEFAULT 1,
  
  created_by      VARCHAR(36) REFERENCES users(id),
  created_at      TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at      TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  
  INDEX idx_org (organization_id),
  INDEX idx_trigger (trigger_type, status)
);

CREATE TABLE workflow_instances (
  id              VARCHAR(36) PRIMARY KEY,
  template_id     VARCHAR(36) NOT NULL REFERENCES workflow_templates(id),
  organization_id VARCHAR(36) NOT NULL REFERENCES organizations(id),
  
  status          ENUM('pending', 'running', 'paused', 'completed', 'failed', 'cancelled') DEFAULT 'pending',
  current_step    INT DEFAULT 0,
  
  input_data      JSON,
  output_data     JSON,
  error           TEXT,
  
  triggered_by    ENUM('user', 'event', 'schedule', 'agent'),
  triggered_by_id VARCHAR(36),
  trigger_event   JSON,
  
  started_at      TIMESTAMP,
  completed_at    TIMESTAMP,
  created_at      TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  
  INDEX idx_org_status (organization_id, status),
  INDEX idx_template (template_id)
);

CREATE TABLE workflow_step_logs (
  id              VARCHAR(36) PRIMARY KEY,
  instance_id     VARCHAR(36) NOT NULL REFERENCES workflow_instances(id),
  
  step_index      INT NOT NULL,
  step_name       VARCHAR(100),
  status          ENUM('pending', 'running', 'completed', 'failed', 'skipped') DEFAULT 'pending',
  
  input_data      JSON,
  output_data     JSON,
  error           TEXT,
  
  started_at      TIMESTAMP,
  completed_at    TIMESTAMP,
  
  INDEX idx_instance (instance_id)
);
```

---

## PART 3: Vector Store Schema

### 3.1 Vector Document Interface

```typescript
interface VectorDocument {
  id: string;                    // UUID
  
  // Tenant isolation
  organization_id: string;       // REQUIRED for filtering
  project_id?: string;
  
  // Content
  content: string;               // Text chunk
  embedding: number[];           // Vector (1536 dims for OpenAI)
  
  // Metadata
  source_type: 'document' | 'issue' | 'task' | 'message';
  source_id: string;             // Original entity ID
  chunk_index: number;           // For long documents
  
  // Timestamps
  created_at: Date;
  updated_at: Date;
}
```

### 3.2 Query Pattern (REQUIRED)

```typescript
// ALWAYS filter by organization_id
const results = await vectorDB.query({
  vector: queryEmbedding,
  filter: {
    organization_id: { $eq: tenantId },  // REQUIRED
    project_id: { $eq: projectId }       // Optional
  },
  topK: 10
});
```

### 3.3 Tenant Isolation Summary

| Entity | Tenant Field | Index |
|--------|--------------|-------|
| projects | organization_id | ✅ |
| issues | organization_id | ✅ |
| tasks | organization_id | ✅ |
| documents | organization_id | ✅ |
| agents | organization_id | ✅ |
| chats | organization_id | ✅ |
| messages | organization_id | ✅ |
| workflow_* | organization_id | ✅ |

> **Rule:** Mọi query PHẢI có filter `organization_id` để đảm bảo tenant isolation.

---

## PART 4: REST API Contracts & DTOs

### 4.1 API Design Principles

```
┌─────────────────────────────────────────────────────────────┐
│                    API DESIGN PRINCIPLES                     │
├─────────────────────────────────────────────────────────────┤
│  ✅ RESTful conventions                                      │
│  ✅ Consistent response format                               │
│  ✅ Proper HTTP status codes                                 │
│  ✅ Tenant context in every request                         │
│  ✅ Pagination for list endpoints                           │
│  ✅ Validation via DTOs                                      │
└─────────────────────────────────────────────────────────────┘
```

### 4.2 Response Formats

#### Success Response

```typescript
interface ApiResponse<T> {
  success: true;
  data: T;
  meta?: {
    pagination?: {
      page: number;
      limit: number;
      total: number;
      totalPages: number;
    };
  };
}
```

#### Error Response

```typescript
interface ApiErrorResponse {
  success: false;
  error: {
    code: string;           // e.g., "VALIDATION_ERROR", "NOT_FOUND"
    message: string;        // Human-readable message
    details?: Record<string, string[]>;  // Field-level errors
  };
}
```

### 4.3 HTTP Status Codes

| Status | Usage |
|--------|-------|
| 200 | Success (GET, PUT, PATCH) |
| 201 | Created (POST) |
| 204 | No Content (DELETE) |
| 400 | Bad Request / Validation Error |
| 401 | Unauthorized |
| 403 | Forbidden |
| 404 | Not Found |
| 409 | Conflict (duplicate) |
| 500 | Internal Server Error |

---

### 4.4 Auth API

**Endpoints:**

| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/api/auth/register` | Register new user |
| POST | `/api/auth/login` | Login |
| POST | `/api/auth/logout` | Logout |
| POST | `/api/auth/refresh` | Refresh token |
| GET | `/api/auth/me` | Get current user  |

**DTOs:**

```typescript
interface RegisterDTO {
  email: string;
  password: string;        // min 8 chars
  displayName: string;
}

interface LoginDTO {
  email: string;
  password: string;
}

interface AuthResponse {
  user: {
    id: string;
    email: string;
    displayName: string;
    avatarUrl?: string;
  };
  tokens: {
    accessToken: string;
    refreshToken: string;
    expiresIn: number;
  };
}
```

---

### 4.5 Organization API

**Endpoints:**

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/api/organizations` | List user's organizations |
| POST | `/api/organizations` | Create organization |
| GET | `/api/organizations/:orgId` | Get organization details |
| PUT | `/api/organizations/:orgId` | Update organization |
| GET | `/api/organizations/:orgId/members` | List members |
| POST | `/api/organizations/:orgId/members` | Invite member |

---

### 4.6 Project API

**Endpoints:**

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/api/org/:orgId/projects` | List projects |
| POST | `/api/org/:orgId/projects` | Create project |
| GET | `/api/org/:orgId/projects/:projectId` | Get project |
| PUT | `/api/org/:orgId/projects/:projectId` | Update project |

---

### 4.7 Issue, Task, Document APIs

See sections 6-8 in original step-5b for full details:
- Issue API: List, create, update, comments
- Task API: List, create, update status
- Document API: List, create, update, versions, embed

---

### 4.8 AI-Specific APIs

#### Agent API

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/api/org/:orgId/agents` | List agents |
| POST | `/api/org/:orgId/agents` | Create agent |
| GET | `/api/org/:orgId/agents/:agentId` | Get agent |
| PUT | `/api/org/:orgId/agents/:agentId` | Update agent |

#### Chat API

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/api/chats` | List chats |
| POST | `/api/chats` | Create new chat |
| GET | `/api/chats/:chatId` | Get chat with messages |
| POST | `/api/chats/:chatId/messages` | Send message |

#### Workflow API

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/api/projects/:projectId/workflows` | List workflow templates |
| POST | `/api/projects/:projectId/workflows` | Create workflow |
| POST | `/api/projects/:projectId/workflows/:wfId/trigger` | Manual trigger |
| GET | `/api/workflow-instances/:instanceId` | Get instance details |

---

## Related Documents

- [Step 4 - Directory Structure](step-4-directory-structure.md)
- [Step 3 - Architecture Design](step-3-architecture.md)
- [Step 2 - MVP Scope](step-2-mvp-scope.md)

---

*Document Version: 2.0 (Consolidated)*  
*Last Updated: 2026-02-11*
