# Step 5: Entity Definition & Database Schema

**Date:** 2026-02-07  
**Status:** ✅ Completed

## Sources

- Previous: [Step 4 - Directory Structure](step-4-directory-structure.md)
- Architecture: [Step 3 - High-Level Architecture](step-3-high-level-architecture.md)

---

## 1. Entity Relationship Diagram (ERD)

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

## 2. Core Entities

### 2.1 User

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

| Field | Type | Description |
|-------|------|-------------|
| id | UUID | Primary key |
| email | String | Unique, login identifier |
| password_hash | String | Bcrypt hashed |
| display_name | String | Display name |
| status | Enum | active/inactive/suspended |

---

### 2.2 Organization

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

| Field | Type | Description |
|-------|------|-------------|
| id | UUID | Primary key |
| slug | String | URL-friendly (e.g., `my-company`) |
| settings | JSON | Org-level configurations |

---

### 2.3 Member (User ↔ Organization)

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

| Field | Type | Description |
|-------|------|-------------|
| role | Enum | owner/admin/member |
| status | Enum | active/pending/removed |

---

### 2.4 Project

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

---

### 2.5 Issue

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

---

### 2.6 Task

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

---

### 2.7 Document

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

-- Document versions for history
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

## 3. AI Entities

### 3.1 Agent

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

---

### 3.2 Chat & Message

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

---

### 3.3 Workflow

```sql
-- Workflow Templates (design)
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

-- Workflow Instances (execution)
CREATE TABLE workflow_instances (
  id              VARCHAR(36) PRIMARY KEY,
  template_id     VARCHAR(36) NOT NULL REFERENCES workflow_templates(id),
  organization_id VARCHAR(36) NOT NULL REFERENCES organizations(id),
  
  status          ENUM('pending', 'running', 'paused', 'completed', 'failed', 'cancelled') DEFAULT 'pending',
  current_step    INT DEFAULT 0,
  
  input_data      JSON,                          -- Input parameters
  output_data     JSON,                          -- Execution results
  error           TEXT,
  
  triggered_by    ENUM('user', 'event', 'schedule', 'agent'),
  triggered_by_id VARCHAR(36),                   -- User ID or Agent ID
  trigger_event   JSON,                          -- Event that triggered
  
  started_at      TIMESTAMP,
  completed_at    TIMESTAMP,
  created_at      TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  
  INDEX idx_org_status (organization_id, status),
  INDEX idx_template (template_id)
);

-- Workflow Step Logs
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

## 4. Vector Store Schema (Pinecone/Weaviate)

```typescript
// Vector DB Schema (conceptual)
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

### Vector Query Pattern

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

---

## 5. Key Constraints Summary

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

## 6. Related Documents

- [Step 4 - Directory Structure](step-4-directory-structure.md)
- [Step 3 - High-Level Architecture](step-3-high-level-architecture.md)
- [Step 3b - Data Flows](step-3b-data-flows.md)
- [Step 2 - MVP Scope](step-2-mvp-scope.md)

---

## 7. GitHub Issues

- #8: Step 5 - Entity Definition & Database Schema

---

*Document Version: 1.0*
*Last Updated: 2026-02-07*
