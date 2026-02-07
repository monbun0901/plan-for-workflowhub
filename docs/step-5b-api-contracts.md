# Step 5b: API Contracts & DTOs

**Date:** 2026-02-07  
**Status:** ✅ Completed

## Sources

- Previous: [Step 5a - Entity & Database Schema](step-5a-entity-database-schema.md)
- Architecture: [Step 4 - Directory Structure](step-4-directory-structure.md)

---

## 1. API Design Principles

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

---

## 2. Base Response Format

### Success Response

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

### Error Response

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

### HTTP Status Codes

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

## 3. Auth API

### Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/api/auth/register` | Register new user |
| POST | `/api/auth/login` | Login |
| POST | `/api/auth/logout` | Logout |
| POST | `/api/auth/refresh` | Refresh token |
| GET | `/api/auth/me` | Get current user |

### DTOs

```typescript
// POST /api/auth/register
interface RegisterDTO {
  email: string;           // required, email format
  password: string;        // required, min 8 chars
  displayName: string;     // required, 2-100 chars
}

// POST /api/auth/login
interface LoginDTO {
  email: string;
  password: string;
}

// Response
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

## 4. Organization API

### Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/api/organizations` | List user's organizations |
| POST | `/api/organizations` | Create organization |
| GET | `/api/organizations/:orgId` | Get organization details |
| PUT | `/api/organizations/:orgId` | Update organization |
| DELETE | `/api/organizations/:orgId` | Delete organization |
| GET | `/api/organizations/:orgId/members` | List members |
| POST | `/api/organizations/:orgId/members` | Invite member |

### DTOs

```typescript
// POST /api/organizations
interface CreateOrganizationDTO {
  name: string;            // required, 2-100 chars
  slug: string;            // required, lowercase, no spaces
  description?: string;
}

// PUT /api/organizations/:orgId
interface UpdateOrganizationDTO {
  name?: string;
  description?: string;
  logoUrl?: string;
  settings?: Record<string, any>;
}

// Response
interface OrganizationResponse {
  id: string;
  name: string;
  slug: string;
  description?: string;
  logoUrl?: string;
  memberCount: number;
  myRole: 'owner' | 'admin' | 'member';
  createdAt: string;
}

// POST /api/organizations/:orgId/members
interface InviteMemberDTO {
  email: string;
  role: 'admin' | 'member';
}
```

---

## 5. Project API

### Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/api/org/:orgId/projects` | List projects |
| POST | `/api/org/:orgId/projects` | Create project |
| GET | `/api/org/:orgId/projects/:projectId` | Get project |
| PUT | `/api/org/:orgId/projects/:projectId` | Update project |
| DELETE | `/api/org/:orgId/projects/:projectId` | Delete project |

### DTOs

```typescript
// POST /api/org/:orgId/projects
interface CreateProjectDTO {
  name: string;            // required
  slug: string;            // required, unique within org
  description?: string;
  visibility?: 'private' | 'internal' | 'public';
}

// Response
interface ProjectResponse {
  id: string;
  organizationId: string;
  name: string;
  slug: string;
  description?: string;
  status: 'active' | 'archived';
  visibility: 'private' | 'internal' | 'public';
  issueCount: number;
  taskCount: number;
  documentCount: number;
  createdBy: { id: string; displayName: string };
  createdAt: string;
  updatedAt: string;
}
```

---

## 6. Issue API

### Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/api/projects/:projectId/issues` | List issues |
| POST | `/api/projects/:projectId/issues` | Create issue |
| GET | `/api/projects/:projectId/issues/:issueNumber` | Get issue |
| PUT | `/api/projects/:projectId/issues/:issueNumber` | Update issue |
| DELETE | `/api/projects/:projectId/issues/:issueNumber` | Delete issue |
| GET | `/api/projects/:projectId/issues/:issueNumber/comments` | Get comments |
| POST | `/api/projects/:projectId/issues/:issueNumber/comments` | Add comment |

### DTOs

```typescript
// POST /api/projects/:projectId/issues
interface CreateIssueDTO {
  title: string;           // required, max 200 chars
  description?: string;
  type?: 'bug' | 'feature' | 'improvement' | 'question';
  priority?: 'low' | 'medium' | 'high' | 'critical';
  assigneeId?: string;
  labels?: string[];
  dueDate?: string;        // ISO date
}

// Query params for GET /api/projects/:projectId/issues
interface ListIssuesQuery {
  status?: 'open' | 'in_progress' | 'resolved' | 'closed';
  type?: string;
  priority?: string;
  assigneeId?: string;
  search?: string;
  page?: number;
  limit?: number;
  sortBy?: 'created' | 'updated' | 'priority';
  sortOrder?: 'asc' | 'desc';
}

// Response
interface IssueResponse {
  id: string;
  projectId: string;
  number: number;
  title: string;
  description?: string;
  type: string;
  status: string;
  priority: string;
  assignee?: { id: string; displayName: string; avatarUrl?: string };
  reporter: { id: string; displayName: string };
  labels: string[];
  dueDate?: string;
  taskCount: number;
  commentCount: number;
  createdAt: string;
  updatedAt: string;
  closedAt?: string;
}
```

---

## 7. Task API

### Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/api/projects/:projectId/tasks` | List tasks |
| POST | `/api/projects/:projectId/tasks` | Create task |
| GET | `/api/projects/:projectId/tasks/:taskId` | Get task |
| PUT | `/api/projects/:projectId/tasks/:taskId` | Update task |
| DELETE | `/api/projects/:projectId/tasks/:taskId` | Delete task |
| PATCH | `/api/projects/:projectId/tasks/:taskId/status` | Update status |

### DTOs

```typescript
// POST /api/projects/:projectId/tasks
interface CreateTaskDTO {
  title: string;           // required
  description?: string;
  issueId?: string;        // Link to issue
  priority?: 'low' | 'medium' | 'high' | 'critical';
  assigneeId?: string;
  estimatedHours?: number;
  dueDate?: string;
}

// PATCH /api/projects/:projectId/tasks/:taskId/status
interface UpdateTaskStatusDTO {
  status: 'todo' | 'in_progress' | 'review' | 'done' | 'cancelled';
  actualHours?: number;    // Required when status = done
}

// Response
interface TaskResponse {
  id: string;
  projectId: string;
  issueId?: string;
  title: string;
  description?: string;
  status: string;
  priority: string;
  assignee?: { id: string; displayName: string };
  createdBy: { id: string; displayName: string };
  estimatedHours?: number;
  actualHours?: number;
  dueDate?: string;
  completedAt?: string;
  aiGenerated: boolean;
  generatedByAgent?: { id: string; name: string };
  createdAt: string;
  updatedAt: string;
}
```

---

## 8. Document API

### Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/api/projects/:projectId/documents` | List documents |
| POST | `/api/projects/:projectId/documents` | Create document |
| GET | `/api/projects/:projectId/documents/:docId` | Get document |
| PUT | `/api/projects/:projectId/documents/:docId` | Update document |
| DELETE | `/api/projects/:projectId/documents/:docId` | Delete document |
| GET | `/api/projects/:projectId/documents/:docId/versions` | Get versions |
| POST | `/api/projects/:projectId/documents/:docId/embed` | Trigger embedding |

### DTOs

```typescript
// POST /api/projects/:projectId/documents
interface CreateDocumentDTO {
  title: string;           // required
  slug: string;            // required, unique in project
  content?: string;        // Markdown
  parentId?: string;       // For nested docs
  status?: 'draft' | 'published';
}

// Response
interface DocumentResponse {
  id: string;
  projectId: string;
  title: string;
  slug: string;
  content: string;
  contentType: 'markdown' | 'rich_text';
  parentId?: string;
  status: string;
  version: number;
  embeddingStatus: 'pending' | 'processing' | 'completed' | 'failed';
  createdBy: { id: string; displayName: string };
  updatedBy?: { id: string; displayName: string };
  createdAt: string;
  updatedAt: string;
}
```

---

## 9. Agent API

### Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/api/org/:orgId/agents` | List agents |
| POST | `/api/org/:orgId/agents` | Create agent |
| GET | `/api/org/:orgId/agents/:agentId` | Get agent |
| PUT | `/api/org/:orgId/agents/:agentId` | Update agent |
| DELETE | `/api/org/:orgId/agents/:agentId` | Delete agent |
| POST | `/api/org/:orgId/agents/:agentId/test` | Test agent |

### DTOs

```typescript
// POST /api/org/:orgId/agents
interface CreateAgentDTO {
  name: string;            // required
  role: 'pm' | 'developer' | 'reviewer' | 'analyst' | 'custom';
  description?: string;
  systemPrompt: string;    // required
  projectId?: string;      // Optional, null = org-wide
  capabilities?: string[]; // Allowed tools/actions
  model?: string;          // Default: gpt-4
  temperature?: number;    // Default: 0.7
}

// Response
interface AgentResponse {
  id: string;
  organizationId: string;
  projectId?: string;
  name: string;
  role: string;
  description?: string;
  systemPrompt: string;
  capabilities: string[];
  status: 'active' | 'disabled';
  model: string;
  temperature: number;
  createdBy: { id: string; displayName: string };
  createdAt: string;
  updatedAt: string;
}
```

---

## 10. Chat API

### Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/api/chats` | List chats |
| POST | `/api/chats` | Create new chat |
| GET | `/api/chats/:chatId` | Get chat with messages |
| DELETE | `/api/chats/:chatId` | Delete chat |
| POST | `/api/chats/:chatId/messages` | Send message |
| GET | `/api/chats/:chatId/messages` | Get messages (paginated) |

### DTOs

```typescript
// POST /api/chats
interface CreateChatDTO {
  agentId?: string;        // Optional, use default agent if null
  projectId?: string;      // Context
  contextType?: 'general' | 'project' | 'issue' | 'task' | 'document';
  contextId?: string;
  title?: string;
}

// POST /api/chats/:chatId/messages
interface SendMessageDTO {
  content: string;         // required
}

// Response
interface MessageResponse {
  id: string;
  chatId: string;
  role: 'user' | 'assistant' | 'system';
  content: string;
  modelUsed?: string;
  tokensInput?: number;
  tokensOutput?: number;
  sources?: {              // RAG sources
    id: string;
    title: string;
    type: 'document' | 'issue' | 'task';
    relevanceScore: number;
  }[];
  createdAt: string;
}

// Chat Response with messages
interface ChatResponse {
  id: string;
  organizationId: string;
  projectId?: string;
  agent?: { id: string; name: string; role: string };
  title: string;
  status: 'active' | 'archived';
  contextType?: string;
  contextId?: string;
  messages: MessageResponse[];
  createdAt: string;
  updatedAt: string;
}
```

---

## 11. Workflow API

### Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/api/projects/:projectId/workflows` | List workflow templates |
| POST | `/api/projects/:projectId/workflows` | Create workflow |
| GET | `/api/projects/:projectId/workflows/:wfId` | Get workflow |
| PUT | `/api/projects/:projectId/workflows/:wfId` | Update workflow |
| DELETE | `/api/projects/:projectId/workflows/:wfId` | Delete workflow |
| POST | `/api/projects/:projectId/workflows/:wfId/trigger` | Manual trigger |
| GET | `/api/projects/:projectId/workflows/:wfId/instances` | List instances |
| GET | `/api/workflow-instances/:instanceId` | Get instance details |

### DTOs

```typescript
// POST /api/projects/:projectId/workflows
interface CreateWorkflowDTO {
  name: string;            // required
  description?: string;
  triggerType: 'manual' | 'event' | 'scheduled' | 'ai';
  triggerConfig?: {
    eventType?: string;    // For event-based
    schedule?: string;     // Cron expression for scheduled
    condition?: object;    // For AI-triggered
  };
  steps: WorkflowStepDTO[];
}

interface WorkflowStepDTO {
  name: string;
  type: 'action' | 'condition' | 'delay' | 'ai_task';
  config: Record<string, any>;
  onSuccess?: number;      // Next step index
  onFailure?: number;
}

// Response
interface WorkflowInstanceResponse {
  id: string;
  templateId: string;
  template: { id: string; name: string };
  status: string;
  currentStep: number;
  inputData?: Record<string, any>;
  outputData?: Record<string, any>;
  error?: string;
  triggeredBy: string;
  startedAt?: string;
  completedAt?: string;
  steps: {
    stepIndex: number;
    stepName: string;
    status: string;
    inputData?: Record<string, any>;
    outputData?: Record<string, any>;
    error?: string;
    startedAt?: string;
    completedAt?: string;
  }[];
  createdAt: string;
}
```

---

## 12. Common Query Parameters

```typescript
// Pagination (all list endpoints)
interface PaginationQuery {
  page?: number;           // Default: 1
  limit?: number;          // Default: 20, max: 100
}

// Sorting (where applicable)
interface SortQuery {
  sortBy?: string;         // Field name
  sortOrder?: 'asc' | 'desc';
}

// Search (where applicable)
interface SearchQuery {
  search?: string;         // Full-text search
  q?: string;              // Alternative
}
```

---

## 13. Related Documents

- [Step 5a - Entity & Database Schema](step-5a-entity-database-schema.md)
- [Step 4 - Directory Structure](step-4-directory-structure.md)
- [Step 3 - High-Level Architecture](step-3-high-level-architecture.md)

---

## 14. GitHub Issues

- #9: Step 5b - API Contracts & DTOs

---

*Document Version: 1.0*
*Last Updated: 2026-02-07*
