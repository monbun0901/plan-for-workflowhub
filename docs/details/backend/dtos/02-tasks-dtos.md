# Tasks DTOs

**Module:** Tasks  
**Path:** `apps/api/src/modules/tasks/dtos/`

---

## 📋 Database Schema Reference

```sql
CREATE TABLE tasks (
  id, issue_id, project_id, organization_id,
  title, description,
  status_id, priority, visibility_id,
  created_by, 
  estimated_hours, actual_hours, due_date, completed_at,
  ai_generated, generated_by_agent,
  created_at, updated_at
);
-- Categories via task_categories junction
-- Tags via task_tags junction
-- Assignees via task_assignees junction
```

---

## 1️⃣ CreateTaskDTO

**File:** `create-task.dto.ts`

```typescript
import { z } from 'zod';

/**
 * Create Task DTO
 * @description Validation schema for creating a new task
 */
export const CreateTaskSchema = z.object({
  title: z.string().min(1, 'Title is required').max(200, 'Title too long'),
  description: z.string().optional(),
  
  // References
  project_id: z.string().uuid('Invalid project ID'),
  issue_id: z.string().uuid('Invalid issue ID').optional(),
  status_id: z.string().uuid('Invalid status ID').optional(),
  
  // Priority ENUM
  priority: z.enum(['low', 'medium', 'high', 'critical']).default('medium'),
  visibility_id: z.string().uuid('Invalid visibility ID').optional(),
  
  // Multi-select via junction tables
  category_ids: z.array(z.string().uuid('Invalid category ID')).optional(),
  tag_ids: z.array(z.string().uuid('Invalid tag ID')).optional(),
  assignee_ids: z.array(z.string().uuid('Invalid assignee ID')).optional(),
  
  // Time tracking
  estimated_hours: z.number()
    .min(0, 'Estimated hours must be positive')
    .max(999.99, 'Estimated hours too large')
    .optional(),
  due_date: z.string().datetime('Invalid datetime format').optional(),
});

export type CreateTaskDTO = z.infer<typeof CreateTaskSchema>;
```

**Notes:**
- `organization_id` - Set from route param or inherited from project
- `created_by` - Set from `req.user.id`
- `ai_generated`, `generated_by_agent` - Set when created by AI workflow

---

## 2️⃣ UpdateTaskDTO

**File:** `update-task.dto.ts`

```typescript
import { z } from 'zod';
import { CreateTaskSchema } from './create-task.dto';

/**
 * Update Task DTO
 * @description All fields optional except project_id (immutable)
 */
export const UpdateTaskSchema = CreateTaskSchema
  .partial()
  .omit({ project_id: true });

export type UpdateTaskDTO = z.infer<typeof UpdateTaskSchema>;
```

**Notes:**
- Cannot change `project_id` after creation (prevents cross-project moves)
- `completed_at` auto-set when `status.state_type` → 'done'

---

## 3️⃣ UpdateTaskHoursDTO

**File:** `update-task-hours.dto.ts`

```typescript
import { z } from 'zod';

/**
 * Update Task Hours DTO
 * @description Dedicated endpoint for time tracking updates
 */
export const UpdateTaskHoursSchema = z.object({
  actual_hours: z.number()
    .min(0, 'Actual hours must be positive')
    .max(999.99, 'Actual hours too large'),
});

export type UpdateTaskHoursDTO = z.infer<typeof UpdateTaskHoursSchema>;
```

**Use case:** `PATCH /:orgId/tasks/:id/hours` for time tracker integrations

---

## 4️⃣ TaskResponseDTO

**File:** `task-response.dto.ts`

```typescript
/**
 * Task Response DTO
 * @description Shape of task data returned to client
 */
export interface TaskResponseDTO {
  id: string;
  project_id: string;
  issue_id: string | null;
  organization_id: string;
  
  title: string;
  description: string | null;
  
  status: {
    id: string;
    name: string;
    state_type: 'todo' | 'in_progress' | 'done' | 'closed';
    color?: string;
  } | null;
  
  priority: 'low' | 'medium' | 'high' | 'critical';
  
  visibility: {
    id: string;
    name: string;
  } | null;
  
  // Multi-select populations
  categories: Array<{
    id: string;
    name: string;
    color?: string;
  }>;
  
  tags: Array<{
    id: string;
    name: string;
    color?: string;
  }>;
  
  assignees: Array<{
    id: string;
    name: string;
    email: string;
    avatar: string | null;
  }>;
  
  created_by: {
    id: string;
    name: string;
    avatar: string | null;
  } | null;
  
  // Time tracking
  estimated_hours: number | null;
  actual_hours: number | null;
  due_date: string | null; // ISO 8601
  completed_at: string | null; // ISO 8601
  
  // AI metadata (Phase 2)
  ai_generated: boolean;
  generated_by_agent: {
    id: string;
    name: string;
  } | null;
  
  created_at: string; // ISO 8601
  updated_at: string; // ISO 8601
}
```

---

## 5️⃣ TaskFilterDTO

**File:** `task-filter.dto.ts`

```typescript
import { z } from 'zod';

/**
 * Task Filter DTO
 * @description Query parameters for GET /tasks
 */
export const TaskFilterSchema = z.object({
  // Pagination
  page: z.coerce.number().min(1).default(1),
  limit: z.coerce.number().min(1).max(100).default(20),
  
  // Search
  search: z.string().optional(),
  
  // Filters
  project_id: z.string().uuid().optional(),
  issue_id: z.string().uuid().optional(),
  status: z.string().uuid().optional(),
  priority: z.enum(['low', 'medium', 'high', 'critical']).optional(),
  assignees: z.array(z.string().uuid()).or(z.string().uuid()).optional(),
  categories: z.array(z.string().uuid()).or(z.string().uuid()).optional(),
  tags: z.array(z.string().uuid()).or(z.string().uuid()).optional(),
  
  // Date filters
  due_before: z.string().date().optional(),
  due_after: z.string().date().optional(),
  completed: z.coerce.boolean().optional(), // true = has completed_at
  
  // Sorting
  sort: z.enum([
    'newest', 'oldest', 'priority', 'due_date', 'updated', 'title'
  ]).default('newest'),
});

export type TaskFilterDTO = z.infer<typeof TaskFilterSchema>;
```

---

## 📡 API Endpoints

| Method | Endpoint | DTO Used |
|--------|----------|----------|
| `POST` | `/:orgId/projects/:projectId/tasks` | CreateTaskDTO |
| `GET` | `/:orgId/tasks` | TaskFilterDTO (query) |
| `GET` | `/:orgId/tasks/:id` | - |
| `PATCH` | `/:orgId/tasks/:id` | UpdateTaskDTO |
| `PATCH` | `/:orgId/tasks/:id/hours` | UpdateTaskHoursDTO |
| `DELETE` | `/:orgId/tasks/:id` | - |

---

## 🔒 Validation Rules

1. **Project Validation**: `project_id` must exist and belong to `organization_id`
2. **Issue Validation**: If `issue_id` provided, must belong to same `project_id`
3. **Status Validation**: `status_id` must exist with `target_type='task'`
4. **Assignee Validation**: All `assignee_ids` must be members of organization
5. **Due Date Validation**: Cannot be in the past (optional warning)
6. **Completion Logic**: When status → 'done', auto-set `completed_at = NOW()`

---

*Last Updated: 2026-02-12*
