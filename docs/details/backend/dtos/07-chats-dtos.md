# Chats DTOs

**Module:** Chats  
**Path:** `apps/api/src/modules/chats/dtos/`

---

## 📋 Database Schema Reference

```sql
CREATE TABLE chats (
  id, organization_id, project_id,
  user_id, agent_id,
  title, status,
  context_type, context_id,
  created_at, updated_at
);
-- status: ENUM('active', 'archived')
-- context_type: ENUM('general', 'project', 'issue', 'task', 'document')
```

---

## 1️⃣ CreateChatDTO

**File:** `create-chat.dto.ts`

```typescript
import { z } from 'zod';

/**
 * Create Chat DTO
 * @description Validation schema for creating a new AI chat session
 */
export const CreateChatSchema = z.object({
  title: z.string()
    .min(1, 'Title is required')
    .max(200, 'Title too long')
    .optional(), // Auto-generated from first message if not provided
  
  agent_id: z.string().uuid('Invalid agent ID'),
  project_id: z.string().uuid('Invalid project ID').optional(),
  
  // Context (what is this chat about?)
  context_type: z.enum([
    'general', 
    'project', 
    'issue', 
    'task', 
    'document'
  ]).default('general'),
  
  context_id: z.string().uuid('Invalid context ID').optional(),
}).refine(
  data => {
    // If context_type is not 'general', context_id is required
    if (data.context_type !== 'general' && !data.context_id) {
      return false;
    }
    return true;
  },
  {
    message: 'context_id is required when context_type is not "general"',
    path: ['context_id'],
  }
);

export type CreateChatDTO = z.infer<typeof CreateChatSchema>;
```

**Auto-generated fields:**
- `user_id` - Set from `req.user.id`
- `title` - Auto-generated from first message if not provided
- `status` - Defaults to 'active'

---

## 2️⃣ SendMessageDTO

**File:** `send-message.dto.ts`

```typescript
import { z } from 'zod';

/**
 * Send Message DTO
 * @description Validation schema for sending a message in a chat
 */
export const SendMessageSchema = z.object({
  content: z.string()
    .min(1, 'Message cannot be empty')
    .max(10000, 'Message too long'),
  
  // Attachments can reference existing entities
  attachments: z.array(z.object({
    type: z.enum(['document', 'code', 'image']),
    reference_id: z.string().uuid('Invalid reference ID'),
  })).optional(),
});

export type SendMessageDTO = z.infer<typeof SendMessageSchema>;
```

**Server-side workflow:**
1. Validate chat exists and belongs to user
2. Insert user message into `messages` table
3. Call AI agent with message + context
4. Insert AI response into `messages` table
5. Update chat `updated_at`
6. Return both messages

---

## 3️⃣ UpdateChatDTO

**File:** `update-chat.dto.ts`

```typescript
import { z } from 'zod';

/**
 * Update Chat DTO
 * @description Only title and status can be updated
 */
export const UpdateChatSchema = z.object({
  title: z.string().min(1).max(200).optional(),
  status: z.enum(['active', 'archived']).optional(),
});

export type UpdateChatDTO = z.infer<typeof UpdateChatSchema>;
```

---

## 4️⃣ ChatResponseDTO

**File:** `chat-response.dto.ts`

```typescript
/**
 * Chat Response DTO
 * @description Shape of chat data returned to client
 */
export interface ChatResponseDTO {
  id: string;
  organization_id: string;
  project_id: string | null;
  
  user: {
    id: string;
    name: string;
    avatar: string | null;
  };
  
  agent: {
    id: string;
    name: string;
    role: 'pm' | 'developer' | 'reviewer' | 'analyst' | 'custom';
  };
  
  title: string;
  status: 'active' | 'archived';
  
  // Context
  context: {
    type: 'general' | 'project' | 'issue' | 'task' | 'document';
    id: string | null;
    label?: string; // Populated entity name (e.g., "Fix login bug")
  };
  
  // Latest message preview
  last_message?: {
    content: string; // Truncated
    sender: 'user' | 'agent';
    created_at: string;
  };
  
  message_count: number;
  
  created_at: string; // ISO 8601
  updated_at: string; // ISO 8601
}
```

---

## 5️⃣ ChatFilterDTO

**File:** `chat-filter.dto.ts`

```typescript
import { z } from 'zod';

/**
 * Chat Filter DTO
 * @description Query parameters for GET /chats
 */
export const ChatFilterSchema = z.object({
  // Pagination
  page: z.coerce.number().min(1).default(1),
  limit: z.coerce.number().min(1).max(100).default(20),
  
  // Search
  search: z.string().optional(),
  
  // Filters
  project_id: z.string().uuid().optional(),
  agent_id: z.string().uuid().optional(),
  status: z.enum(['active', 'archived']).optional(),
  context_type: z.enum([
    'general', 'project', 'issue', 'task', 'document'
  ]).optional(),
  
  // Sorting
  sort: z.enum(['newest', 'oldest', 'updated']).default('updated'),
});

export type ChatFilterDTO = z.infer<typeof ChatFilterSchema>;
```

---

## 📡 API Endpoints

| Method | Endpoint | DTO Used |
|--------|----------|----------|
| `POST` | `/:orgId/chats` | CreateChatDTO |
| `POST` | `/:orgId/chats/:id/messages` | SendMessageDTO |
| `GET` | `/:orgId/chats` | ChatFilterDTO (query) |
| `GET` | `/:orgId/chats/:id` | - (includes messages) |
| `PATCH` | `/:orgId/chats/:id` | UpdateChatDTO |
| `DELETE` | `/:orgId/chats/:id` | - (sets status='archived') |

---

## 🔒 Validation Rules

1. **Agent Validation**: `agent_id` must exist and be active
2. **Context Validation**: If `context_type` != 'general', validate `context_id` exists
3. **Project Scope**: If `project_id` provided, agent must be org-wide or belong to same project
4. **Authorization**: Users can only access their own chats

---

*Last Updated: 2026-02-12*
