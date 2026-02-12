# Backend DTOs - README

**Directory:** `docs/details/backend/dtos/`  
**Last Updated:** 2026-02-12

---

## 📚 Available DTO Specifications

| # | Module | File | DTOs Included |
|---|--------|------|---------------|
| 1 | Projects | [01-projects-dtos.md](./01-projects-dtos.md) | Create, Update, Response, Filter |
| 2 | Tasks | [02-tasks-dtos.md](./02-tasks-dtos.md) | Create, Update, UpdateHours, Response, Filter |
| 3 | Issues | [03-issues-dtos.md](./03-issues-dtos.md) | Create, Update, Response, Filter |
| 4 | Documents | [04-documents-dtos.md](./04-documents-dtos.md) | Create, Update, Response, Filter |
| 5 | Organizations | [05-organizations-dtos.md](./05-organizations-dtos.md) | Create, Update, Response, List |
| 6 | Members | [06-members-dtos.md](./06-members-dtos.md) | Invite, Update, AcceptInvite, Response, Filter |
| 7 | Chats | [07-chats-dtos.md](./07-chats-dtos.md) | Create, SendMessage, Update, Response, Filter |
| 8 | Agents | [08-agents-dtos.md](./08-agents-dtos.md) | Create, Update, Response, Filter |

---

## 🎯 DTO Pattern Overview

Each specification follows a consistent structure:

### 1. **CreateDTO**
- Validation for `POST` requests
- All required fields for entity creation
- Excludes auto-generated fields (id, timestamps)

### 2. **UpdateDTO**
- Validation for `PATCH` requests
- All fields optional (partial update)
- Immutable fields excluded (id, org_id, created_by)

### 3. **ResponseDTO**
- Shape of data returned to client
- Includes auto-generated fields
- Populated relationships (nested objects)

### 4. **FilterDTO**
- Query parameters for list endpoints
- Pagination (page, limit)
- Search & filters
- Sorting options

---

## 🔧 Technology Stack

### Validation Library
**Zod** - TypeScript-first schema validation

```typescript
import { z } from 'zod';

const schema = z.object({
  name: z.string().min(1).max(100),
  email: z.string().email(),
});

type DTO = z.infer<typeof schema>;
```

### Controller Integration
```typescript
// Validate request body
const validated = CreateProjectSchema.parse(req.body);

// Auto-throws ZodError if invalid
```

---

## 📋 Common Validation Patterns

### Strings
```typescript
z.string()
  .min(1, 'Required')
  .max(100, 'Too long')
  .regex(/^[a-z0-9-]+$/, 'Invalid format')
```

### UUIDs
```typescript
z.string().uuid('Invalid ID')
```

### Arrays
```typescript
z.array(z.string().uuid()).optional()
```

### Enums
```typescript
z.enum(['low', 'medium', 'high', 'critical'])
```

### Dates
```typescript
z.string().datetime() // ISO 8601
z.string().date()     // YYYY-MM-DD
```

---

## 🔗 Related Documentation

- [Database Tables](../database/tables/) - Source schemas for DTOs
- [04-projects-module.md](../backend/04-projects-module.md) - Reference implementation
- [18-api-routes-map.md](../backend/18-api-routes-map.md) - API endpoint mapping

---

## 🚀 Implementation Guide

### 1. Create DTO Files
```bash
apps/api/src/modules/{entity}/dtos/
├── create-{entity}.dto.ts
├── update-{entity}.dto.ts
├── {entity}-response.dto.ts
└── {entity}-filter.dto.ts
```

### 2. Import in Controller
```typescript
import { CreateProjectSchema } from '../dtos/create-project.dto';

export class ProjectController {
  async create(req, res) {
    const validated = CreateProjectSchema.parse(req.body);
    // ...
  }
}
```

### 3. Error Handling
```typescript
try {
  const validated = schema.parse(data);
} catch (error) {
  if (error instanceof z.ZodError) {
    return res.status(422).json({
      success: false,
      error: {
        code: 'VALIDATION_ERROR',
        details: error.errors,
      },
    });
  }
}
```

---

## ✅ Checklist for Adding New DTOs

- [ ] Review database schema in `database/tables/`
- [ ] Create DTO file in `backend/dtos/`
- [ ] Define Create, Update, Response, Filter DTOs
- [ ] Add validation rules with error messages
- [ ] Document auto-generated fields
- [ ] List API endpoints
- [ ] Add validation rules section
- [ ] Update this README index

---

*Generated: 2026-02-12*
