# API Endpoints Constants

**Version:** v1.0.0
**Status:** 📦 Single Source of Truth
**Path:** `src/constants/api-endpoints.ts`

---

## 🎯 Purpose

Tập trung toàn bộ API routes vào một file duy nhất. Không service nào được phép hard-code URL trực tiếp.

---

## 📝 Cấu trúc

```typescript
// src/constants/api-endpoints.ts

/** Base API URL từ environment */
const API_BASE = process.env.NEXT_PUBLIC_API_URL;

export const API_ENDPOINTS = {
  AUTH: {
    LOGIN: `${API_BASE}/auth/login`,
    REGISTER: `${API_BASE}/auth/register`,
    REFRESH: `${API_BASE}/auth/refresh`,
    LOGOUT: `${API_BASE}/auth/logout`,
  },
  USERS: {
    ME: `${API_BASE}/users/me`,
    UPDATE_ME: `${API_BASE}/users/me`,
    PREFERENCES: `${API_BASE}/users/me/preferences`,
  },
  ORGANIZATIONS: {
    LIST: `${API_BASE}/organizations`,
    CREATE: `${API_BASE}/organizations`,
    DETAIL: (orgId: string) => `${API_BASE}/organizations/${orgId}`,
    UPDATE: (orgId: string) => `${API_BASE}/organizations/${orgId}`,
    MEMBERS: (orgId: string) => `${API_BASE}/organizations/${orgId}/members`,
    INVITE_MEMBER: (orgId: string) => `${API_BASE}/organizations/${orgId}/members/invite`,
  },
  PROJECTS: {
    LIST: (orgId: string) => `${API_BASE}/${orgId}/projects`,
    DETAIL: (orgId: string, id: string) => `${API_BASE}/${orgId}/projects/${id}`,
    CREATE: (orgId: string) => `${API_BASE}/${orgId}/projects`,
    UPDATE: (orgId: string, id: string) => `${API_BASE}/${orgId}/projects/${id}`,
    DELETE: (orgId: string, id: string) => `${API_BASE}/${orgId}/projects/${id}`,
    STATS: (orgId: string, id: string) => `${API_BASE}/${orgId}/projects/${id}/stats`,
  },
  TASKS: {
    LIST: (orgId: string, projectId: string) => `${API_BASE}/${orgId}/projects/${projectId}/tasks`,
    DETAIL: (orgId: string, projectId: string, taskId: string) => 
      `${API_BASE}/${orgId}/projects/${projectId}/tasks/${taskId}`,
    CREATE: (orgId: string, projectId: string) => `${API_BASE}/${orgId}/projects/${projectId}/tasks`,
    UPDATE: (orgId: string, projectId: string, taskId: string) =>
      `${API_BASE}/${orgId}/projects/${projectId}/tasks/${taskId}`,
    UPDATE_STATUS: (orgId: string, taskId: string) => `${API_BASE}/${orgId}/tasks/${taskId}/status`,
    ASSIGN: (orgId: string, taskId: string) => `${API_BASE}/${orgId}/tasks/${taskId}/assign`,
  },
  ISSUES: {
    LIST: (orgId: string, projectId: string) => `${API_BASE}/${orgId}/projects/${projectId}/issues`,
    LIST_ALL: (orgId: string) => `${API_BASE}/${orgId}/issues`,
    DETAIL: (orgId: string, projectId: string, issueId: string) =>
      `${API_BASE}/${orgId}/projects/${projectId}/issues/${issueId}`,
    CREATE: (orgId: string, projectId: string) => `${API_BASE}/${orgId}/projects/${projectId}/issues`,
  },
  LOOKUPS: {
    WORKFLOW_STATUSES: (orgId: string, targetType?: 'project' | 'issue' | 'task' | 'document' | 'workflow_template') =>
      `${API_BASE}/${orgId}/lookups/workflow-statuses${targetType ? `?target_type=${targetType}` : ''}`,
    CATEGORIES: (orgId: string) => `${API_BASE}/${orgId}/lookups/categories`,
    TAGS: (orgId: string) => `${API_BASE}/${orgId}/lookups/tags`,
    ROLES: (orgId: string) => `${API_BASE}/${orgId}/lookups/roles`,
  },
} as const;
```

---

## 🔑 Rules

- **Dùng hàm cho dynamic routes:** `DETAIL: (id) => ...` thay vì string interpolation rải rác.
- **`as const`:** Đảm bảo TypeScript infer chính xác kiểu.
- **Environment-based:** `API_BASE` lấy từ `.env`, không hard-code domain.

---

## 📚 Related
- [03-app-routes.md](03-app-routes.md) — Frontend routes
- [../../03-api-service.md](../../03-api-service.md) — API service sử dụng endpoints

---

*Last Updated: 2026-02-11*
