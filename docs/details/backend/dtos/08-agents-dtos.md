# Agents DTOs

**Module:** Agents  
**Path:** `apps/api/src/modules/agents/dtos/`

---

## 📋 Database Schema Reference

```sql
CREATE TABLE agents (
  id, organization_id, project_id,
  name, role, description,
  system_prompt, model, tools (JSON), document_ids (JSON),
  status,
  created_at, updated_at
);
-- role: ENUM('pm', 'developer', 'reviewer', 'analyst', 'custom')
-- status: ENUM('active', 'disabled')
```

---

## 1️⃣ CreateAgentDTO

**File:** `create-agent.dto.ts`

```typescript
import { z } from 'zod';

/**
 * Create Agent DTO
 * @description Validation schema for creating a new AI agent
 */
export const CreateAgentSchema = z.object({
  name: z.string().min(1, 'Name is required').max(100, 'Name too long'),
  role: z.enum(['pm', 'developer', 'reviewer', 'analyst', 'custom']),
  description: z.string().optional(),
  project_id: z.string().uuid('Invalid project ID').optional(), // NULL = org-wide agent
  
  // AI Configuration
  system_prompt: z.string()
    .min(1, 'System prompt is required')
    .max(5000, 'System prompt too long'),
  model: z.string().default('gpt-4'),
  
  // Tools (array of tool names)
  tools: z.array(z.string()).optional(),
  
  // Document IDs for RAG context
  document_ids: z.array(z.string().uuid('Invalid document ID')).optional(),
});

export type CreateAgentDTO = z.infer<typeof CreateAgentSchema>;
```

**Auto-generated fields:**
- `status` - Defaults to 'active'

**Notes:**
- `tools` - Array of tool names (e.g., `['web_search', 'code_execution', 'file_access']`)
- `document_ids` - Documents to include in agent's context for RAG

---

## 2️⃣ UpdateAgentDTO

**File:** `update-agent.dto.ts`

```typescript
import { z } from 'zod';
import { CreateAgentSchema } from './create-agent.dto';

/**
 * Update Agent DTO
 * @description All fields optional except project_id (immutable)
 */
export const UpdateAgentSchema = CreateAgentSchema
  .partial()
  .omit({ project_id: true });

export type UpdateAgentDTO = z.infer<typeof UpdateAgentSchema>;
```

**Notes:**
- Cannot change `project_id` (scope is fixed)
- Can toggle `status` between 'active' and 'disabled'

---

## 3️⃣ AgentResponseDTO

**File:** `agent-response.dto.ts`

```typescript
/**
 * Agent Response DTO
 * @description Shape of agent data returned to client
 */
export interface AgentResponseDTO {
  id: string;
  organization_id: string;
  project_id: string | null; // NULL = org-wide agent
  
  name: string;
  role: 'pm' | 'developer' | 'reviewer' | 'analyst' | 'custom';
  description: string | null;
  
  // AI Configuration
  system_prompt: string;
  model: string;
  tools: string[] | null; // ['web_search', 'code_execution', etc.]
  
  // RAG context
  documents: Array<{
    id: string;
    title: string;
  }> | null;
  
  status: 'active' | 'disabled';
  
  // Usage stats (optional, for analytics)
  stats?: {
    chat_count: number;
    message_count: number;
    last_used_at: string | null;
  };
  
  created_at: string; // ISO 8601
  updated_at: string; // ISO 8601
}
```

---

## 4️⃣ AgentFilterDTO

**File:** `agent-filter.dto.ts`

```typescript
import { z } from 'zod';

/**
 * Agent Filter DTO
 * @description Query parameters for GET /agents
 */
export const AgentFilterSchema = z.object({
  // Pagination
  page: z.coerce.number().min(1).default(1),
  limit: z.coerce.number().min(1).max(100).default(20),
  
  // Search
  search: z.string().optional(),
  
  // Filters
  project_id: z.string().uuid().optional(),
  role: z.enum(['pm', 'developer', 'reviewer', 'analyst', 'custom']).optional(),
  status: z.enum(['active', 'disabled']).optional(),
  
  // Sorting
  sort: z.enum(['newest', 'oldest', 'name', 'role']).default('newest'),
});

export type AgentFilterDTO = z.infer<typeof AgentFilterSchema>;
```

---

## 📡 API Endpoints

| Method | Endpoint | DTO Used |
|--------|----------|----------|
| `POST` | `/:orgId/agents` | CreateAgentDTO |
| `POST` | `/:orgId/projects/:projectId/agents` | CreateAgentDTO |
| `GET` | `/:orgId/agents` | AgentFilterDTO (query) |
| `GET` | `/:orgId/agents/:id` | - |
| `PATCH` | `/:orgId/agents/:id` | UpdateAgentDTO |
| `DELETE` | `/:orgId/agents/:id` | - (sets status='disabled') |

---

## 🔒 Validation Rules

1. **Project Scope**: If `project_id` provided, must exist and belong to organization
2. **Document Validation**: All `document_ids` must exist and be accessible
3. **Tool Validation**: All tools must be in allowed list (security check)
4. **Model Validation**: Model must be supported by AI Gateway
5. **Authorization**: Only ADMIN or OWNER can create/update agents

---

## 🎯 Agent Role Defaults

**Predefined system prompts per role:**

```typescript
const ROLE_PROMPTS = {
  pm: "You are a Project Manager. Help with planning, task breakdown, and timeline estimation.",
  developer: "You are a Developer. Help with code review, debugging, and implementation.",
  reviewer: "You are a Code Reviewer. Provide constructive feedback on code quality and best practices.",
  analyst: "You are a Business Analyst. Help with requirements gathering and documentation.",
  custom: "" // User provides full prompt
};
```

---

*Last Updated: 2026-02-12*
