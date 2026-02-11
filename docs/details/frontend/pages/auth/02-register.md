# Register Page

**Status:** 🟢 Phase 1 (MVP)
**Route:** `/register`
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
│        │  │  Fullname │ │          │
│        │  └──────────┘ │          │
│        │  ┌──────────┐ │          │
│        │  │  Email    │ │          │
│        │  └──────────┘ │          │
│        │  ┌──────────┐ │          │
│        │  │  Password │ │          │
│        │  └──────────┘ │          │
│        │  ┌──────────┐ │          │
│        │  │  Confirm  │ │          │
│        │  └──────────┘ │          │
│        │               │          │
│        │  [Register]   │          │
│        │               │          │
│        │  Login →       │          │
│        └──────────────┘          │
│                                  │
└──────────────────────────────────┘
```

---

## ✅ Chức năng

| Feature | Status | Mô tả |
|---------|--------|--------|
| Email/Password register | 🟢 MVP | Form validation (Zod) |
| Password strength indicator | 🟢 MVP | Visual feedback |
| Confirm password match | 🟢 MVP | Real-time validation |
| Login link | 🟢 MVP | Navigate to `/login` |
| Terms & Conditions checkbox | 🟡 Scale | Legal compliance |
| OAuth (Google/GitHub) | 🟡 Scale | Social sign-up |

---

## 🧩 Components

| Component | Source |
|-----------|--------|
| `<Input />` | shadcn/ui |
| `<Button />` | shadcn/ui |
| `<FormField />` | shared/forms |
| `<PasswordStrength />` | shared |

## 🪝 Hooks

| Hook | Chức năng |
|------|----------|
| `useRegisterForm()` | Form state + Zod validation |
| `useAuthMutation()` | API call → redirect `/dashboard` + toast |

## 📡 API Endpoints

| Method | Endpoint | Mô tả |
|--------|----------|--------|
| `POST` | `/auth/register` | Tạo tài khoản mới |

## 🗄️ Stores

| Store | Sử dụng |
|-------|---------|
| `useAuthStore` | `setAuth(user)` sau register thành công |

---

*Last Updated: 2026-02-11*
