# Login Page

**Status:** 🟢 Phase 1 (MVP)
**Route:** `/login`
**Layout:** [Auth Layout](../../layouts/02-auth-layout.md)

---

## 📐 Wireframe

```
┌──────────────────────────────────┐
│                                  │
│        ┌──────────────┐          │
│        │   🔷 Logo     │          │
│        │               │          │
│        │  ┌──────────┐ │          │
│        │  │  Email    │ │          │
│        │  └──────────┘ │          │
│        │  ┌──────────┐ │          │
│        │  │  Password │ │          │
│        │  └──────────┘ │          │
│        │               │          │
│        │  [  Login  ]  │          │
│        │               │          │
│        │  ─── or ───   │          │
│        │  [G] Google   │          │
│        │  [H] GitHub   │          │
│        │               │          │
│        │  Forgot pass? │          │
│        │  Register →   │          │
│        └──────────────┘          │
│                                  │
└──────────────────────────────────┘
```

---

## ✅ Chức năng

| Feature | Status | Mô tả |
|---------|--------|--------|
| Email/Password login | 🟢 MVP | Form validation (Zod) |
| Remember me | 🟢 MVP | Checkbox, persist token |
| Forgot password link | 🟢 MVP | Navigate to `/forgot-password` |
| Register link | 🟢 MVP | Navigate to `/register` |
| OAuth (Google) | 🟡 Scale | Social login |
| OAuth (GitHub) | 🟡 Scale | Social login |
| Rate limiting feedback | 🟢 MVP | Toast error sau 5 lần thất bại |

---

## 🧩 Components

| Component | Source |
|-----------|--------|
| `<Input />` | shadcn/ui |
| `<Button />` | shadcn/ui |
| `<FormField />` | shared/forms |
| `<Logo />` | shared |

## 🪝 Hooks

| Hook | Chức năng |
|------|----------|
| `useLoginForm()` | Form state + validation (Zod) |
| `useAuthMutation()` | API call + redirect + toast |

## 📡 API Endpoints

| Method | Endpoint | Mô tả |
|--------|----------|--------|
| `POST` | `/auth/login` | Email + password login |

## 🗄️ Stores

| Store | Sử dụng |
|-------|---------|
| `useAuthStore` | `setAuth(user)` sau login thành công |
| `useTenantStore` | `setTenant(orgId)` nếu user có org |

---

*Last Updated: 2026-02-11*
