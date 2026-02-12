# Issues DTOs

**Module:** Issues  
**Path:** `apps/api/src/modules/issues/dtos/`

---

## 📋 Database Schema Reference

```sql
CREATE TABLE issues (
  id, project_id, organization_id,
  number (INT auto-increment per project),
  title, description,
  status_id, priority, visibility_id,
  reporter_id, due_date,
  created_at, updated_at, closed_at
);
-- Categories via issue_categories junction
-- Tags via issue_tags junction
-- Assignees via issue_assignees junction
```

---

## 1️⃣ CreateIssueDTO

**File:** `create-issue.dto.ts`

```typescript
import { z } from 'zod';

/**
 * Create Issue DTO
 * @description Validation schema for creating a new issue
 */
export const CreateIssueSchema = z.object({
  title: z.string().min(1, 'Title is required').max(200, 'Title too long'),
  description: z.string().optional(),
  
  // References
  project_id: z.string().uuid('Invalid project ID'),
  status_id: z.string().uuid('Invalid status ID').optional(),
  
  // Priority ENUM
  priority: z.enum(['low', 'medium', 'high', 'critical']).default('medium'),
  visibility_id: z.string().uuid('Invalid visibility ID').optional(),
  
  // Multi-select via junction tables
  category_ids: z.array(z.string().uuid('Invalid category ID')).optional(),
  tag_ids: z.array(z.string().uuid('Invalid tag ID')).optional(),
  assignee_ids: z.array(z.string().uuid('Invalid assignee ID')).optional(),
  
  due_date: z.string().date('Invalid date format').optional(),
});

export type CreateIssueDTO = z.infer<typeof CreateIssueSchema>;
```

**Auto-generated fields:**
- `number` - Auto-increment per project (e.g., #1, #2, #3)
- `reporter_id` - Set from `req.user.id`
- `organization_id` - Inherited from project

---

## 2️⃣ UpdateIssueDTO

**File:** `update-issue.dto.ts`

```typescript
import { z } from 'zod';
import { CreateIssueSchema } from './create-issue.dto';

/**
 * Update Issue DTO
 * @description All fields optional except project_id (immutable)
 */
export const UpdateIssueSchema = CreateIssueSchema
  .partial()
  .omit({ project_id: true });

export type UpdateIssueDTO = z.infer<typeof UpdateIssueSchema>;
```

**Notes:**
- `closed_at` auto-set when `status.state_type` → 'closed'
- Cannot change `number` or `reporter_id`

---

## 3️⃣ IssueResponseDTO

**File:** `issue-response.dto.ts`

```typescript
/**
 * Issue Response DTO
 * @description Shape of issue data returned to client
 */
export interface IssueResponseDTO {
  id: string;
  project_id: string;
  organization_id: string;
  number: number; // Auto-incremented per project (#1, #2, etc.)
  
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
  
  reporter: {
    id: string;
    name: string;
    email: string;
    avatar: string | null;
  };
  
  due_date: string | null; // ISO 8601
  
  created_at: string; // ISO 8601
  updated_at: string; // ISO 8601
  closed_at: string | null; // ISO 8601
}
```

---

## 4️⃣ IssueFilterDTO

**File:** `issue-filter.dto.ts`

```typescript
import { z } from 'zod';

/**
 * Issue Filter DTO
 * @description Query parameters for GET /issues
 */
export const IssueFilterSchema = z.object({
  // Pagination
  page: z.coerce.number().min(1).default(1),
  limit: z.coerce.number().min(1).max(100).default(20),
  
  // Search
  search: z.string().optional(),
  
  // Filters
  project_id: z.string().uuid().optional(),
  status: z.string().uuid().optional(),
  priority: z.enum(['low', 'medium', 'high', 'critical']).optional(),
  reporter: z.string().uuid().optional(),
  assignees: z.array(z.string().uuid()).or(z.string().uuid()).optional(),
  categories: z.array(z.string().uuid()).or(z.string().uuid()).optional(),
  tags: z.array(z.string().uuid()).or(z.string().uuid()).optional(),
  
  // Date filters
  due_before: z.string().date().optional(),
  due_after: z.string().date().optional(),
  closed: z.coerce.boolean().optional(), // true = has closed_at
  
  // Sorting
  sort: z.enum([
    'newest', 'oldest', 'priority', 'due_date', 'number', 'updated'
  ]).default('newest'),
});

export type IssueFilterDTO = z.infer<typeof IssueFilterSchema>;
```

---

## 📡 API Endpoints

| Method | Endpoint | DTO Used |
|--------|----------|----------|
| `POST` | `/:orgId/projects/:projectId/issues` | CreateIssueDTO |
| `GET` | `/:orgId/issues` | IssueFilterDTO (query) |
| `GET` | `/:orgId/issues/:id` | - |
| `PATCH` | `/:orgId/issues/:id` | UpdateIssueDTO |
| `DELETE` | `/:orgId/issues/:id` | - |

---

## 🔒 Validation Rules

1. **Project Validation**: `project_id` must exist and belong to `organization_id`
2. **Number Generation**: Auto-increment within project scope
3. **Status Validation**: `status_id` must exist with `target_type='issue'`
4. **Reporter**: Always set from authenticated user
5. **Closure Logic**: When status → 'closed', auto-set `closed_at = NOW()`

---

*Last Updated: 2026-02-12*
