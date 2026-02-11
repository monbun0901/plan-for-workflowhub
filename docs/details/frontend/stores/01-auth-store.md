# Auth Store

**Version:** v1.0.0
**Status:** 🔐 Identity State
**Path:** `src/stores/auth.store.ts`

---

## 🎯 Responsibility

Quản lý **duy nhất** thông tin xác thực và hồ sơ người dùng. Không chứa logic UI hay config.

---

## 📝 Store Spec

```typescript
// src/stores/auth.store.ts
import { create } from 'zustand';
import { persist } from 'zustand/middleware';
import type { User } from '@/types';

interface AuthState {
  user: User | null;
  isAuthenticated: boolean;
  setAuth: (user: User) => void;
  clearAuth: () => void;
}

export const useAuthStore = create<AuthState>()(
  persist(
    (set) => ({
      user: null,
      isAuthenticated: false,
      setAuth: (user) => set({ user, isAuthenticated: true }),
      clearAuth: () => set({ user: null, isAuthenticated: false }),
    }),
    { name: 'auth-storage' }
  )
);
```

---

## 🔑 Rules

- **Persist:** Dùng `persist` middleware để giữ session qua page refresh.
- **Không chứa tokens:** Access/Refresh tokens nên được quản lý bởi HTTP-only cookies hoặc API interceptor, không lưu trong Zustand.
- **SRP:** Chỉ `user` và `isAuthenticated`. Không thêm theme, language, sidebar state.

---

## 📚 Related
- [04-tenant-store.md](04-tenant-store.md) — Organization context
- [../../03-api-service.md](../../03-api-service.md) — API interceptor (token management)

---

*Last Updated: 2026-02-11*
