# Members DTOs

**Module:** Members  
**Path:** `apps/api/src/modules/members/dtos/`

---

## 📋 Database Schema Reference

```sql
CREATE TABLE members (
  id, user_id, organization_id,
  role_id, status, invited_by, joined_at,
  created_at
);
-- status: ENUM('active', 'pending', 'removed')
```

---

## 1️⃣ InviteMemberDTO

**File:** `invite-member.dto.ts`

```typescript
import { z } from 'zod';

/**
 * Invite Member DTO
 * @description Validation schema for inviting a new member
 */
export const InviteMemberSchema = z.object({
  email: z.string().email('Invalid email format'),
  role_id: z.string().uuid('Invalid role ID'),
  message: z.string()
    .max(500, 'Message too long')
    .optional(), // Optional invitation message
});

export type InviteMemberDTO = z.infer<typeof InviteMemberSchema>;
```

**Server-side workflow:**
1. Check if user exists by email
2. If not, create user account (status='inactive')
3. Create member record (status='pending')
4. Set `invited_by = req.user.id`
5. Send invitation email with accept link
6. Return pending member

---

## 2️⃣ UpdateMemberDTO

**File:** `update-member.dto.ts`

```typescript
import { z } from 'zod';

/**
 * Update Member DTO
 * @description Only role can be updated (status changes via dedicated endpoints)
 */
export const UpdateMemberSchema = z.object({
  role_id: z.string().uuid('Invalid role ID'),
});

export type UpdateMemberDTO = z.infer<typeof UpdateMemberSchema>;
```

**Authorization:** Only OWNER or ADMIN can change roles

---

## 3️⃣ AcceptInviteDTO

**File:** `accept-invite.dto.ts`

```typescript
import { z } from 'zod';

/**
 * Accept Invite DTO
 * @description Validation schema for accepting invitation
 */
export const AcceptInviteSchema = z.object({
  token: z.string().min(1, 'Token is required'), // JWT token from email
});

export type AcceptInviteDTO = z.infer<typeof AcceptInviteSchema>;
```

**Server-side workflow:**
1. Verify JWT token (contains member_id)
2. Update member: status='active', joined_at=NOW()
3. If user.status='inactive', activate user
4. Return success + redirect to organization

---

## 4️⃣ MemberResponseDTO

**File:** `member-response.dto.ts`

```typescript
/**
 * Member Response DTO
 * @description Shape of member data returned to client
 */
export interface MemberResponseDTO {
  id: string;
  
  user: {
    id: string;
    name: string;
    email: string;
    avatar: string | null;
  };
  
  organization_id: string;
  
  role: {
    id: string;
    name: string;
    permissions: string[]; // ['projects.create', 'tasks.edit', etc.]
  };
  
  status: 'active' | 'pending' | 'removed';
  
  invited_by: {
    id: string;
    name: string;
  } | null;
  
  joined_at: string | null; // ISO 8601
  created_at: string; // ISO 8601
}
```

---

## 5️⃣ MemberFilterDTO

**File:** `member-filter.dto.ts`

```typescript
import { z } from 'zod';

/**
 * Member Filter DTO
 * @description Query parameters for GET /members
 */
export const MemberFilterSchema = z.object({
  // Pagination
  page: z.coerce.number().min(1).default(1),
  limit: z.coerce.number().min(1).max(100).default(20),
  
  // Search
  search: z.string().optional(), // Search in user name/email
  
  // Filters
  role: z.string().uuid().optional(),
  status: z.enum(['active', 'pending', 'removed']).optional(),
  
  // Sorting
  sort: z.enum(['newest', 'oldest', 'name', 'role']).default('newest'),
});

export type MemberFilterDTO = z.infer<typeof MemberFilterSchema>;
```

---

## 📡 API Endpoints

| Method | Endpoint | DTO Used |
|--------|----------|----------|
| `POST` | `/:orgId/members/invite` | InviteMemberDTO |
| `POST` | `/members/accept` | AcceptInviteDTO (no orgId - token-based) |
| `GET` | `/:orgId/members` | MemberFilterDTO (query) |
| `GET` | `/:orgId/members/:id` | - |
| `PATCH` | `/:orgId/members/:id` | UpdateMemberDTO |
| `DELETE` | `/:orgId/members/:id` | - (sets status='removed') |

---

## 🔒 Validation Rules

1. **Email Uniqueness**: Check `UNIQUE KEY unique_user_org (user_id, organization_id)`
2. **Role Validation**: `role_id` must exist in organization's roles
3. **Authorization**:
   - INVITE: ADMIN or OWNER only
   - UPDATE_ROLE: ADMIN or OWNER only (cannot change own role)
   - REMOVE: ADMIN or OWNER only (cannot remove self)
4. **Status Transitions**:
   - `pending` → `active`: Via accept invite
   - `active` → `removed`: Via DELETE endpoint
   - Cannot revert `removed`

---

*Last Updated: 2026-02-12*
