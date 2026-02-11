# App Routes Constants

**Version:** v1.0.0
**Status:** 📦 Single Source of Truth
**Path:** `src/constants/routes.ts`

---

## 🎯 Purpose

Tập trung toàn bộ frontend route paths. Không component nào được phép hard-code đường dẫn (ví dụ: `router.push('/projects')` ❌).

---

## 📝 Cấu trúc

```typescript
// src/constants/routes.ts
export const APP_ROUTES = {
  HOME: '/',

  AUTH: {
    LOGIN: '/login',
    REGISTER: '/register',
    FORGOT_PASSWORD: '/forgot-password',
  },

  DASHBOARD: '/dashboard',

  PROJECTS: {
    LIST: '/projects',
    NEW: '/projects/new',
    DETAIL: (id: string) => `/projects/${id}`,
    EDIT: (id: string) => `/projects/${id}/edit`,
    TASKS: (id: string) => `/projects/${id}/tasks`,
    ISSUES: (id: string) => `/projects/${id}/issues`,
    DOCUMENTS: (id: string) => `/projects/${id}/documents`,
  },

  SETTINGS: '/settings',
  MEMBERS: '/members',
  CHAT: '/chat',
} as const;
```

---

## 📝 Usage

```tsx
// ✅ Đúng: Dùng constant
router.push(APP_ROUTES.PROJECTS.NEW);
router.push(APP_ROUTES.PROJECTS.DETAIL(projectId));

// ❌ Sai: Hard-code string
router.push('/projects/new');
router.push(`/projects/${projectId}`);
```

---

## 🔑 Rules

- **Mọi `router.push()`, `<Link href>` phải dùng `APP_ROUTES`.**
- **Thêm route mới:** Khai báo ở đây trước, sau đó tạo page file.
- **Dynamic routes:** Dùng function `(id: string) => ...`.

---

## 📚 Related
- [02-api-endpoints.md](02-api-endpoints.md) — API endpoints (backend routes)
- [01-navigation-config.md](01-navigation-config.md) — Navigation dùng href từ đây

---

*Last Updated: 2026-02-11*
