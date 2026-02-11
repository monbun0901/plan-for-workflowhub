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
  PROJECTS: {
    LIST: `${API_BASE}/projects`,
    DETAIL: (id: string) => `${API_BASE}/projects/${id}`,
    CREATE: `${API_BASE}/projects`,
    UPDATE: (id: string) => `${API_BASE}/projects/${id}`,
    DELETE: (id: string) => `${API_BASE}/projects/${id}`,
  },
  TASKS: {
    LIST: (projectId: string) => `${API_BASE}/projects/${projectId}/tasks`,
    DETAIL: (projectId: string, taskId: string) =>
      `${API_BASE}/projects/${projectId}/tasks/${taskId}`,
  },
  ISSUES: {
    LIST: (projectId: string) => `${API_BASE}/projects/${projectId}/issues`,
    DETAIL: (projectId: string, issueId: string) =>
      `${API_BASE}/projects/${projectId}/issues/${issueId}`,
  },
  ORGANIZATIONS: {
    LIST: `${API_BASE}/organizations`,
    DETAIL: (id: string) => `${API_BASE}/organizations/${id}`,
    MEMBERS: (id: string) => `${API_BASE}/organizations/${id}/members`,
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
