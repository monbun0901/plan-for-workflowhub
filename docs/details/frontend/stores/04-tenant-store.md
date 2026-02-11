# Tenant Store

**Version:** v1.0.0
**Status:** 🏢 Organization Context State
**Path:** `src/stores/tenant.store.ts`

---

## 🎯 Responsibility

Quản lý **duy nhất** bối cảnh tổ chức (organization) hiện tại mà người dùng đang hoạt động.

---

## 📝 Store Spec

```typescript
// src/stores/tenant.store.ts
import { create } from 'zustand';
import { persist } from 'zustand/middleware';

interface TenantState {
  /** ID của organization hiện tại */
  currentOrgId: string | null;
  /** Tên hiển thị (cache để tránh fetch lại) */
  currentOrgName: string | null;
  /** Role của user trong org hiện tại */
  currentRole: string | null;

  setTenant: (orgId: string, orgName: string, role: string) => void;
  clearTenant: () => void;
}

export const useTenantStore = create<TenantState>()(
  persist(
    (set) => ({
      currentOrgId: null,
      currentOrgName: null,
      currentRole: null,

      setTenant: (orgId, orgName, role) =>
        set({ currentOrgId: orgId, currentOrgName: orgName, currentRole: role }),
      clearTenant: () =>
        set({ currentOrgId: null, currentOrgName: null, currentRole: null }),
    }),
    { name: 'tenant-storage' }
  )
);
```

---

## 🔑 Rules

- **Persist:** Giữ tenant context qua sessions (người dùng không phải chọn lại org sau mỗi lần refresh).
- **SRP:** Chỉ chứa org context. Permissions cụ thể nên được tính toán trong hook riêng (ví dụ: `usePermissions`).
- **API Header:** `currentOrgId` được inject vào mọi API request thông qua Axios interceptor.

---

## 📚 Related
- [01-auth-store.md](01-auth-store.md) — User identity (ai đang đăng nhập)
- [../../03-api-service.md](../../03-api-service.md) — Interceptor inject `X-Org-Id` header

---

*Last Updated: 2026-02-11*
