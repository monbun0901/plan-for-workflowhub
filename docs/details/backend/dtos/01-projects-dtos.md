# Projects DTOs

**Module:** Projects  
**Path:** `apps/api/src/modules/projects/dtos/`

---

## 📋 Database Schema Reference

```sql
CREATE TABLE projects (
  id, organization_id, 
  name, slug, description,
  status_id, category_id, visibility_id,
  settings (JSON), created_by,
  created_at, updated_at
);
-- Tags via project_tags junction table
```

---

## 1️⃣ CreateProjectDTO

**File:** `create-project.dto.ts`

```typescript
import { z } from 'zod';

/**
 * Create Project DTO
 * @description Validation schema for creating a new project
 */
export const CreateProjectSchema = z.object({
  name: z.string().min(1, 'Name is required').max(100, 'Name too long'),
  slug: z.string()
    .min(1, 'Slug is required')
    .max(100, 'Slug too long')
    .regex(/^[a-z0-9-]+$/, 'Slug must be lowercase alphanumeric with hyphens'),
  description: z.string().optional(),
  
  // Dynamic lookups (nullable for initial creation)
  status_id: z.string().uuid('Invalid status ID').optional(),
  category_id: z.string().uuid('Invalid category ID').optional(),
  visibility_id: z.string().uuid('Invalid visibility ID').optional(),
  
  // Tags (many-to-many via junction table)
  tag_ids: z.array(z.string().uuid('Invalid tag ID')).optional(),
  
  // JSON settings
  settings: z.record(z.unknown()).optional(),
});

export type CreateProjectDTO = z.infer<typeof CreateProjectSchema>;
```

**Notes:**
- `organization_id` - Set from route param `/:orgId/`
- `created_by` - Set from `req.user.id`
- `slug` - Can be auto-generated from `name` if not provided

---

## 2️⃣ UpdateProjectDTO

**File:** `update-project.dto.ts`

```typescript
import { z } from 'zod';
import { CreateProjectSchema } from './create-project.dto';

/**
 * Update Project DTO
 * @description All fields optional (partial update)
 */
export const UpdateProjectSchema = CreateProjectSchema.partial();

export type UpdateProjectDTO = z.infer<typeof UpdateProjectSchema>;
```

**Notes:**
- Cannot change `organization_id` (tenant isolation)
- Cannot change `id` (immutable primary key)

---

## 3️⃣ ProjectResponseDTO

**File:** `project-response.dto.ts`

```typescript
/**
 * Project Response DTO
 * @description Shape of project data returned to client
 */
export interface ProjectResponseDTO {
  id: string;
  organization_id: string;
  
  name: string;
  slug: string;
  description: string | null;
  
  // Populated relationships
  status: {
    id: string;
    name: string;
    state_type: 'todo' | 'in_progress' | 'done' | 'closed';
    color?: string;
  } | null;
  
  category: {
    id: string;
    name: string;
    color?: string;
  } | null;
  
  visibility: {
    id: string;
    name: string;
    level: string;
  } | null;
  
  tags: Array<{
    id: string;
    name: string;
    color?: string;
  }>;
  
  created_by: {
    id: string;
    name: string;
    avatar: string | null;
  };
  
  settings: Record<string, any> | null;
  
  created_at: string; // ISO 8601
  updated_at: string; // ISO 8601
}
```

---

## 4️⃣ ProjectFilterDTO

**File:** `project-filter.dto.ts`

```typescript
import { z } from 'zod';

/**
 * Project Filter DTO
 * @description Query parameters for GET /projects
 */
export const ProjectFilterSchema = z.object({
  // Pagination
  page: z.coerce.number().min(1).default(1),
  limit: z.coerce.number().min(1).max(100).default(20),
  
  // Search
  search: z.string().optional(), // Search in name, description
  
  // Filters
  status: z.string().uuid().optional(),
  category: z.string().uuid().optional(),
  visibility: z.string().uuid().optional(),
  tags: z.array(z.string().uuid()).or(z.string().uuid()).optional(),
  
  // Sorting
  sort: z.enum(['newest', 'oldest', 'name_asc', 'name_desc', 'updated']).default('newest'),
});

export type ProjectFilterDTO = z.infer<typeof ProjectFilterSchema>;
```

**Query string examples:**
```
GET /api/:orgId/projects?page=1&limit=20
GET /api/:orgId/projects?search=auth&status=uuid
GET /api/:orgId/projects?tags=uuid1,uuid2&sort=newest
```

---

## 📡 API Endpoints

| Method | Endpoint | DTO Used |
|--------|----------|----------|
| `POST` | `/:orgId/projects` | CreateProjectDTO |
| `GET` | `/:orgId/projects` | ProjectFilterDTO (query) |
| `GET` | `/:orgId/projects/:id` | - |
| `PATCH` | `/:orgId/projects/:id` | UpdateProjectDTO |
| `DELETE` | `/:orgId/projects/:id` | - |

---

## 🔒 Validation Rules

1. **Unique Slug**: Check `UNIQUE KEY unique_org_slug (organization_id, slug)`
2. **Tenant Isolation**: All queries MUST filter by `organization_id`
3. **Status Validation**: If `status_id` provided, must exist in `workflow_statuses` with `target_type='project'`
4. **Category Validation**: If `category_id` provided, must exist in `categories` for org
5. **Tag Validation**: All `tag_ids` must exist in `tags` for org

---

*Last Updated: 2026-02-12*
