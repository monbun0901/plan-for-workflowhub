# Documents DTOs

**Module:** Documents  
**Path:** `apps/api/src/modules/documents/dtos/`

---

## 📋 Database Schema Reference

```sql
CREATE TABLE documents (
  id, project_id, organization_id,
  title, slug, content, content_type,
  parent_id, order_index,
  status_id, visibility_id, version,
  created_by, updated_by,
  embedding_status, last_embedded_at,
  created_at, updated_at
);
-- Categories via document_categories junction
-- Tags via document_tags junction
-- Collaborators via document_collaborators junction
```

---

## 1️⃣ CreateDocumentDTO

**File:** `create-document.dto.ts`

```typescript
import { z } from 'zod';

/**
 * Create Document DTO
 * @description Validation schema for creating a new document
 */
export const CreateDocumentSchema = z.object({
  title: z.string().min(1, 'Title is required').max(200, 'Title too long'),
  slug: z.string()
    .min(1, 'Slug is required')
    .max(200, 'Slug too long')
    .regex(/^[a-z0-9-]+$/, 'Slug must be lowercase alphanumeric with hyphens'),
  content: z.string().optional(),
  content_type: z.enum(['markdown', 'rich_text']).default('markdown'),
  
  // References
  project_id: z.string().uuid('Invalid project ID').optional(), // NULL = org-level doc
  status_id: z.string().uuid('Invalid status ID').optional(),
  visibility_id: z.string().uuid('Invalid visibility ID').optional(),
  
  // Hierarchy (for nested docs)
  parent_id: z.string().uuid('Invalid parent ID').optional(),
  order_index: z.number().int().min(0).default(0),
  
  // Multi-select via junction tables
  category_ids: z.array(z.string().uuid('Invalid category ID')).optional(),
  tag_ids: z.array(z.string().uuid('Invalid tag ID')).optional(),
  collaborator_ids: z.array(z.string().uuid('Invalid collaborator ID')).optional(),
});

export type CreateDocumentDTO = z.infer<typeof CreateDocumentSchema>;
```

**Auto-generated fields:**
- `version` - Starts at 1
- `embedding_status` - Defaults to 'pending' (triggers RAG pipeline)
- `created_by` - Set from `req.user.id`

---

## 2️⃣ UpdateDocumentDTO

**File:** `update-document.dto.ts`

```typescript
import { z } from 'zod';
import { CreateDocumentSchema } from './create-document.dto';

/**
 * Update Document DTO
 * @description All fields optional (partial update)
 */
export const UpdateDocumentSchema = CreateDocumentSchema.partial();

export type UpdateDocumentDTO = z.infer<typeof UpdateDocumentSchema>;
```

**Notes:**
- Updating `content` increments `version`
- `updated_by` set from `req.user.id`
- `embedding_status` → 'pending' when content changes (re-index)

---

## 3️⃣ DocumentResponseDTO

**File:** `document-response.dto.ts`

```typescript
/**
 * Document Response DTO
 * @description Shape of document data returned to client
 */
export interface DocumentResponseDTO {
  id: string;
  project_id: string | null; // NULL = org-level document
  organization_id: string;
  
  title: string;
  slug: string;
  content: string | null;
  content_type: 'markdown' | 'rich_text';
  
  // Hierarchy
  parent: {
    id: string;
    title: string;
  } | null;
  order_index: number;
  
  status: {
    id: string;
    name: string;
    state_type: 'todo' | 'in_progress' | 'done' | 'closed';
  } | null;
  
  visibility: {
    id: string;
    name: string;
  } | null;
  
  // Multi-select populations
  categories: Array<{
    id: string;
    name: string;
  }>;
  
  tags: Array<{
    id: string;
    name: string;
  }>;
  
  collaborators: Array<{
    id: string;
    name: string;
    email: string;
    avatar: string | null;
  }>;
  
  // Versioning
  version: number;
  
  // RAG status (system-managed)
  embedding_status: 'pending' | 'processing' | 'completed' | 'failed';
  last_embedded_at: string | null; // ISO 8601
  
  created_by: {
    id: string;
    name: string;
  };
  
  updated_by: {
    id: string;
    name: string;
  } | null;
  
  created_at: string; // ISO 8601
  updated_at: string; // ISO 8601
}
```

---

## 4️⃣ DocumentFilterDTO

**File:** `document-filter.dto.ts`

```typescript
import { z } from 'zod';

/**
 * Document Filter DTO
 * @description Query parameters for GET /documents
 */
export const DocumentFilterSchema = z.object({
  // Pagination
  page: z.coerce.number().min(1).default(1),
  limit: z.coerce.number().min(1).max(100).default(20),
  
  // Search
  search: z.string().optional(), // Search in title, content
  
  // Filters
  project_id: z.string().uuid().optional(),
  status: z.string().uuid().optional(),
  categories: z.array(z.string().uuid()).or(z.string().uuid()).optional(),
  tags: z.array(z.string().uuid()).or(z.string().uuid()).optional(),
  collaborators: z.array(z.string().uuid()).or(z.string().uuid()).optional(),
  parent_id: z.string().uuid().optional(), // Get children of specific doc
  
  // Sorting
  sort: z.enum([
    'newest', 'oldest', 'title', 'updated', 'version'
  ]).default('newest'),
});

export type DocumentFilterDTO = z.infer<typeof DocumentFilterSchema>;
```

---

## 📡 API Endpoints

| Method | Endpoint | DTO Used |
|--------|----------|----------|
| `POST` | `/:orgId/documents` | CreateDocumentDTO |
| `POST` | `/:orgId/projects/:projectId/documents` | CreateDocumentDTO |
| `GET` | `/:orgId/documents` | DocumentFilterDTO (query) |
| `GET` | `/:orgId/documents/:id` | - |
| `PATCH` | `/:orgId/documents/:id` | UpdateDocumentDTO |
| `DELETE` | `/:orgId/documents/:id` | - |

---

## 🔒 Validation Rules

1. **Unique Slug**: Check `UNIQUE KEY unique_project_slug (project_id, slug)`
2. **Parent Validation**: If `parent_id` provided, must exist and have same `project_id`
3. **Status Validation**: `status_id` must exist with `target_type='document'`
4. **Collaborator Validation**: All `collaborator_ids` must be members of organization
5. **Version Increment**: Auto-increment when `content` or `content_type` changes
6. **RAG Trigger**: Set `embedding_status='pending'` on content updates

---

*Last Updated: 2026-02-12*
