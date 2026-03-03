# 📡 API Reference — WorkflowHub

> **Version:** v1 · **Base URL:** `http://localhost:5000/api/v1` · **Auth:** Bearer Token (JWT)

## Mục Lục

- [1. Response Format](#1-response-format)
- [2. Authentication](#2-authentication)
- [3. Auth Endpoints](#3-auth-endpoints)
- [4. Projects](#4-projects)
- [5. Tasks](#5-tasks)
- [6. Issues](#6-issues)
- [7. Documents](#7-documents)
- [8. Users](#8-users)
- [9. Categories](#9-categories)
- [10. Statuses](#10-statuses)
- [11. Upload](#11-upload)
- [12. Workflows](#12-workflows)
- [13. AI — Agents](#13-ai--agents)
- [14. AI — Chat](#14-ai--chat)
- [15. Error Codes](#15-error-codes)

---

## 1. Response Format

### Success Response

```json
{
  "success": true,
  "message": "Success",
  "data": { ... },
  "meta": {
    "page": 1,
    "limit": 20,
    "total": 150,
    "totalPages": 8
  }
}
```

### Error Response

```json
{
  "success": false,
  "message": "Validation failed",
  "code": "VALIDATION_ERROR",
  "errors": [
    { "field": "email", "message": "Invalid email", "code": "invalid_string" }
  ]
}
```

---

## 2. Authentication

Tất cả protected routes yêu cầu header:

```
Authorization: Bearer <access_token>
```

Token flow:
1. `POST /auth/login` → nhận `accessToken` + `refreshToken`
2. Dùng `accessToken` cho mỗi request
3. Khi token hết hạn → `POST /auth/refresh` với `refreshToken`

---

## 3. Auth Endpoints

| Method | Endpoint | Auth | RBAC | Mô tả |
|--------|----------|------|------|-------|
| `POST` | `/auth/register` | ❌ | - | Đăng ký tài khoản |
| `POST` | `/auth/login` | ❌ | - | Đăng nhập |
| `POST` | `/auth/refresh` | ❌ | - | Refresh access token |
| `GET` | `/auth/me` | ✅ | - | Lấy profile user hiện tại |

### `POST /auth/register`

**Body:**
```json
{
  "email": "user@example.com",
  "password": "securePassword123",
  "display_name": "John Doe"
}
```

### `POST /auth/login`

**Body:**
```json
{
  "email": "user@example.com",
  "password": "securePassword123"
}
```

**Response:**
```json
{
  "success": true,
  "data": {
    "user": { "id": "...", "email": "...", "display_name": "...", "role": { ... } },
    "accessToken": "eyJ...",
    "refreshToken": "eyJ..."
  }
}
```

### `POST /auth/refresh`

**Body:**
```json
{
  "refreshToken": "eyJ..."
}
```

---

## 4. Projects

| Method | Endpoint | Auth | RBAC | Mô tả |
|--------|----------|------|------|-------|
| `GET` | `/projects` | ✅ | - | List projects |
| `GET` | `/projects/:projectId` | ✅ | - | Get project detail |
| `POST` | `/projects` | ✅ | admin, manager | Create project |
| `PATCH` | `/projects/:projectId` | ✅ | project admin/manager | Update project |
| `DELETE` | `/projects/:projectId` | ✅ | project admin | Delete project |
| `GET` | `/projects/:projectId/members` | ✅ | - | List project members |
| `POST` | `/projects/:projectId/members` | ✅ | project admin/manager | Add member |
| `PATCH` | `/projects/:projectId/members/:userId` | ✅ | project admin/manager | Update member role |
| `DELETE` | `/projects/:projectId/members/:userId` | ✅ | project admin/manager | Remove member |

> **Note:** `:projectId` có thể là UUID, `key`, hoặc `slug`.

---

## 5. Tasks

| Method | Endpoint | Auth | RBAC | Mô tả |
|--------|----------|------|------|-------|
| `GET` | `/tasks` | ✅ | - | List tasks (với filters) |
| `GET` | `/tasks/:taskId` | ✅ | - | Get task detail |
| `POST` | `/tasks` | ✅ | - | Create task |
| `PATCH` | `/tasks/:taskId` | ✅ | - | Update task |
| `DELETE` | `/tasks/:taskId` | ✅ | - | Delete task |

### Query Parameters (list)

| Param | Type | Mô tả |
|-------|------|-------|
| `project_id` | string | Filter by project |
| `status_id` | string | Filter by status |
| `assignee_id` | string | Filter by assignee |
| `page` | number | Page number (default: 1) |
| `limit` | number | Items per page (default: 20) |
| `sort` | string | Sort field |
| `order` | string | `asc` \| `desc` |

---

## 6. Issues

| Method | Endpoint | Auth | RBAC | Mô tả |
|--------|----------|------|------|-------|
| `GET` | `/issues` | ✅ | - | List issues |
| `GET` | `/issues/:issueId` | ✅ | - | Get issue detail |
| `POST` | `/issues` | ✅ | - | Create issue |
| `PATCH` | `/issues/:issueId` | ✅ | - | Update issue |
| `DELETE` | `/issues/:issueId` | ✅ | - | Delete issue |

---

## 7. Documents

| Method | Endpoint | Auth | RBAC | Mô tả |
|--------|----------|------|------|-------|
| `GET` | `/documents` | ✅ | - | List documents |
| `GET` | `/documents/:docId` | ✅ | - | Get document detail |
| `POST` | `/documents` | ✅ | - | Create document |
| `PATCH` | `/documents/:docId` | ✅ | - | Update document |
| `DELETE` | `/documents/:docId` | ✅ | - | Delete document |

---

## 8. Users

| Method | Endpoint | Auth | RBAC | Mô tả |
|--------|----------|------|------|-------|
| `GET` | `/users` | ✅ | - | List users |
| `GET` | `/users/roles` | ✅ | - | List available roles |
| `GET` | `/users/:userId` | ✅ | - | Get user detail |
| `POST` | `/users` | ✅ | admin, manager | Create user account |
| `PATCH` | `/users/:userId` | ✅ | admin, manager | Update user / change role |
| `DELETE` | `/users/:userId` | ✅ | admin, manager | Delete user |

---

## 9. Categories

| Method | Endpoint | Auth | RBAC | Mô tả |
|--------|----------|------|------|-------|
| `GET` | `/categories?type=task` | ✅ | - | List categories by type |
| `GET` | `/categories/:id` | ✅ | - | Get category detail |
| `POST` | `/categories` | ✅ | admin, manager | Create category |
| `PATCH` | `/categories/:id` | ✅ | admin, manager | Update category |
| `DELETE` | `/categories/:id` | ✅ | admin only | Delete category |

### Query Parameters

| Param | Type | Values |
|-------|------|--------|
| `type` | string | `project` \| `task` \| `issue` \| `document` \| `workflow` \| `member` |

---

## 10. Statuses

| Method | Endpoint | Auth | RBAC | Mô tả |
|--------|----------|------|------|-------|
| `GET` | `/statuses?type=task` | ✅ | - | List statuses by type |
| `GET` | `/statuses/:id` | ✅ | - | Get status detail |
| `POST` | `/statuses` | ✅ | - | Create status |
| `PATCH` | `/statuses/:id` | ✅ | - | Update status |
| `DELETE` | `/statuses/:id` | ✅ | - | Delete status |

### Query Parameters

| Param | Type | Values |
|-------|------|--------|
| `type` | string | `project` \| `task` \| `issue` \| `document` |

---

## 11. Upload

| Method | Endpoint | Auth | RBAC | Mô tả |
|--------|----------|------|------|-------|
| `POST` | `/upload/image` | ✅ | - | Upload image |
| `DELETE` | `/upload/image` | ✅ | - | Delete image by publicId |

### Upload Image

**Content-Type:** `multipart/form-data`

| Field | Type | Mô tả |
|-------|------|-------|
| `file` | File | Image file |
| `folder` | string (query) | Target folder: `avatars` \| `agents` \| `general` |

---

## 12. Workflows

| Method | Endpoint | Auth | RBAC | Mô tả |
|--------|----------|------|------|-------|
| `GET` | `/workflows` | ✅ | - | List workflow templates |
| `GET` | `/workflows/:id` | ✅ | - | Get template detail |
| `POST` | `/workflows` | ✅ | - | Create template |
| `PATCH` | `/workflows/:id` | ✅ | - | Update template |
| `DELETE` | `/workflows/:id` | ✅ | - | Delete template |
| `POST` | `/workflows/:id/run` | ✅ | - | Execute workflow |
| `GET` | `/workflows/instances` | ✅ | - | List workflow instances |
| `GET` | `/workflows/instances/:id` | ✅ | - | Get instance detail |
| `POST` | `/workflows/instances/:id/approve` | ✅ | - | Approve pending step |
| `GET` | `/workflow-statuses` | ✅ | - | List workflow statuses |

---

## 13. AI — Agents

| Method | Endpoint | Auth | RBAC | Mô tả |
|--------|----------|------|------|-------|
| `GET` | `/agents` | ✅ | - | List agents |
| `GET` | `/agents/providers` | ✅ | - | List available AI providers |
| `GET` | `/agents/:agentId` | ✅ | - | Get agent detail |
| `POST` | `/agents` | ✅ | - | Create agent |
| `PATCH` | `/agents/:agentId` | ✅ | - | Update agent |
| `DELETE` | `/agents/:agentId` | ✅ | - | Delete agent |
| `POST` | `/agents/:agentId/execute` | ✅ | - | Execute agent (one-shot) |
| `POST` | `/agents/:agentId/clone` | ✅ | - | Clone agent |
| `GET` | `/agents/:agentId/documents` | ✅ | - | List agent's knowledge docs |
| `POST` | `/agents/:agentId/documents` | ✅ | - | Add documents to agent |
| `DELETE` | `/agents/:agentId/documents/:documentId` | ✅ | - | Remove document from agent |

---

## 14. AI — Chat

| Method | Endpoint | Auth | RBAC | Mô tả |
|--------|----------|------|------|-------|
| `GET` | `/chats` | ✅ | - | List conversations |
| `GET` | `/chats/:chatId` | ✅ | - | Get conversation detail |
| `POST` | `/chats` | ✅ | - | Create conversation |
| `GET` | `/chats/:chatId/messages` | ✅ | - | List messages |
| `POST` | `/chats/:chatId/messages` | ✅ | - | Send message (get AI response) |
| `DELETE` | `/chats/:chatId` | ✅ | - | Delete conversation |

---

## 15. Error Codes

| HTTP Status | Code | Class | Mô tả |
|-------------|------|-------|-------|
| 400 | `BAD_REQUEST` | `BadRequestError` | Request data không hợp lệ |
| 401 | `UNAUTHORIZED` | `UnauthorizedError` | Chưa xác thực / token invalid |
| 403 | `FORBIDDEN` | `ForbiddenError` | Không đủ quyền |
| 404 | `NOT_FOUND` | `NotFoundError` | Resource không tồn tại |
| 409 | `CONFLICT` | `ConflictError` | Duplicate / xung đột dữ liệu |
| 422 | `VALIDATION_ERROR` | `ValidationError` | Validation thất bại (Zod) |
| 429 | `TOO_MANY_REQUESTS` | `TooManyRequestsError` | Rate limit exceeded |
| 500 | `INTERNAL_ERROR` | - | Lỗi server không xác định |

### Sequelize Error Mapping

| Sequelize Error | HTTP | Code |
|----------------|------|------|
| `SequelizeValidationError` | 422 | `VALIDATION_ERROR` |
| `SequelizeUniqueConstraintError` | 409 | `CONFLICT` |
| `SequelizeForeignKeyConstraintError` | 400 | `BAD_REQUEST` |

---

> **Xem thêm:**
> - [03 — Database Schema](./03-database-schema.md)
> - [05 — Backend Architecture](./05-backend-architecture.md)
> - [08 — RBAC & Permissions](./08-rbac-permissions.md)
