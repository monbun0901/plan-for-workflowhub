# Organizations DTOs

**Module:** Organizations  
**Path:** `apps/api/src/modules/organizations/dtos/`

---

## 📋 Database Schema Reference

```sql
CREATE TABLE organizations (
  id, name, slug, description, logo_url,
  settings (JSON), created_by,
  created_at, updated_at
);
```

---

## 1️⃣ CreateOrganizationDTO

**File:** `create-organization.dto.ts`

```typescript
import { z } from 'zod';

/**
 * Create Organization DTO
 * @description Validation schema for creating a new organization
 */
export const CreateOrganizationSchema = z.object({
  name: z.string().min(1, 'Name is required').max(100, 'Name too long'),
  slug: z.string()
    .min(1, 'Slug is required')
    .max(100, 'Slug too long')
    .regex(/^[a-z0-9-]+$/, 'Slug must be lowercase alphanumeric with hyphens'),
  description: z.string().optional(),
  logo_url: z.string()
    .url('Invalid URL format')
    .max(500, 'URL too long')
    .optional(),
  settings: z.record(z.unknown()).optional(),
});

export type CreateOrganizationDTO = z.infer<typeof CreateOrganizationSchema>;
```

**Auto-generated fields:**
- `created_by` - Set from `req.user.id`
- Creator auto becomes **OWNER** via `members` table insert

**Post-creation workflow:**
1. Insert into `organizations`
2. Insert creator into `members` with role='owner', status='active'
3. Seed default master data (categories, tags, statuses, roles, visibility levels)

---

## 2️⃣ UpdateOrganizationDTO

**File:** `update-organization.dto.ts`

```typescript
import { z } from 'zod';
import { CreateOrganizationSchema } from './create-organization.dto';

/**
 * Update Organization DTO
 * @description All fields optional (partial update)
 */
export const UpdateOrganizationSchema = CreateOrganizationSchema.partial();

export type UpdateOrganizationDTO = z.infer<typeof UpdateOrganizationSchema>;
```

**Authorization:** Only OWNER or ADMIN can update organization

---

## 3️⃣ OrganizationResponseDTO

**File:** `organization-response.dto.ts`

```typescript
/**
 * Organization Response DTO
 * @description Shape of organization data returned to client
 */
export interface OrganizationResponseDTO {
  id: string;
  name: string;
  slug: string;
  description: string | null;
  logo_url: string | null;
  settings: Record<string, any> | null;
  
  created_by: {
    id: string;
    name: string;
    email: string;
    avatar: string | null;
  };
  
  // Aggregate stats (optional, for dashboard)
  stats?: {
    member_count: number;
    project_count: number;
  };
  
  created_at: string; // ISO 8601
  updated_at: string; // ISO 8601
}
```

---

## 4️⃣ OrganizationListResponseDTO

**File:** `organization-list-response.dto.ts`

```typescript
/**
 * Organization List Response DTO
 * @description Simplified shape for list view
 */
export interface OrganizationListItemDTO {
  id: string;
  name: string;
  slug: string;
  logo_url: string | null;
  
  // User's role in this org
  my_role: {
    id: string;
    name: string;
  };
  
  member_count: number;
}
```

---

## 📡 API Endpoints

| Method | Endpoint | DTO Used |
|--------|----------|----------|
| `POST` | `/organizations` | CreateOrganizationDTO |
| `GET` | `/organizations` | - (list user's orgs) |
| `GET` | `/organizations/:orgId` | - |
| `PATCH` | `/organizations/:orgId` | UpdateOrganizationDTO |
| `DELETE` | `/organizations/:orgId` | - (soft delete or OWNER only) |

**Note:** No `orgId` prefix for org-level routes (org is the resource itself)

---

## 🔒 Validation Rules

1. **Unique Slug**: Check global uniqueness (not tenant-scoped)
2. **Owner Creation**: Creator auto-assigned OWNER role
3. **Seed Data**: Auto-create default master data for new org
4. **Authorization**: 
   - CREATE: Any authenticated user
   - UPDATE: OWNER or ADMIN only
   - DELETE: OWNER only (requires confirmation)

---

*Last Updated: 2026-02-12*
