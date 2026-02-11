# UI Tokens Constants

**Version:** v1.0.0
**Status:** 📦 Single Source of Truth
**Path:** `src/constants/ui-tokens.ts`

---

## 🎯 Purpose

Chứa các giá trị hiển thị (labels, colors, mapping) mà cần dùng trong logic UI. Không hard-code trong components.

---

## 📝 Status Labels & Colors

```typescript
// src/constants/ui-tokens.ts

/** Project status mapping */
export const PROJECT_STATUS = {
  ACTIVE: { labelKey: 'status.active', color: 'var(--color-success)' },
  ARCHIVED: { labelKey: 'status.archived', color: 'var(--color-muted)' },
  PAUSED: { labelKey: 'status.paused', color: 'var(--color-warning)' },
} as const;

/** Task priority mapping */
export const TASK_PRIORITY = {
  LOW: { labelKey: 'priority.low', color: 'var(--color-info)', order: 1 },
  MEDIUM: { labelKey: 'priority.medium', color: 'var(--color-warning)', order: 2 },
  HIGH: { labelKey: 'priority.high', color: 'var(--color-danger)', order: 3 },
  CRITICAL: { labelKey: 'priority.critical', color: 'var(--color-destructive)', order: 4 },
} as const;

/** User roles */
export const USER_ROLES = {
  OWNER: { labelKey: 'role.owner', level: 3 },
  ADMIN: { labelKey: 'role.admin', level: 2 },
  MEMBER: { labelKey: 'role.member', level: 1 },
} as const;

/** Pagination defaults */
export const PAGINATION = {
  DEFAULT_PAGE_SIZE: 20,
  PAGE_SIZE_OPTIONS: [10, 20, 50, 100],
} as const;
```

---

## 🔑 Rules

- **Color dùng `var()`:** Không viết hex/rgb trực tiếp, tham chiếu CSS Variables.
- **Label dùng l10n key:** `labelKey` phải là translation key.
- **`as const`:** Đảm bảo type safety.
- **Không có logic:** File này chỉ chứa data, không có hàm xử lý.

---

## 📚 Related
- [01-navigation-config.md](01-navigation-config.md) — Navigation config
- [../patterns/01-localization.md](../patterns/01-localization.md) — l10n system

---

*Last Updated: 2026-02-11*
