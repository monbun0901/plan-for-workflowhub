# Navigation Config Constants

**Version:** v1.0.0
**Status:** 📦 Single Source of Truth
**Path:** `src/constants/navigation.ts`

---

## 🎯 Purpose

File này là **nguồn duy nhất** (Single Source of Truth) cho toàn bộ cấu trúc menu của ứng dụng. Không component nào được phép tự định nghĩa menu riêng.

---

## 📝 Cấu trúc dữ liệu

```typescript
// src/constants/navigation.ts
import {
  LayoutDashboard,
  FolderKanban,
  ListTodo,
  Bug,
  FileText,
  Users,
  Settings,
  MessageSquare,
} from 'lucide-react';
import type { NavItemConfig } from '@/types/navigation';

/** Main sidebar navigation */
export const DASHBOARD_NAV: NavItemConfig[] = [
  {
    titleKey: 'nav.overview',
    href: '/dashboard',
    icon: LayoutDashboard,
    roles: ['owner', 'admin', 'member'],
  },
  {
    titleKey: 'nav.projects',
    href: '/projects',
    icon: FolderKanban,
  },
  {
    titleKey: 'nav.tasks',
    href: '/tasks',
    icon: ListTodo,
  },
  {
    titleKey: 'nav.issues',
    href: '/issues',
    icon: Bug,
  },
  {
    titleKey: 'nav.documents',
    href: '/documents',
    icon: FileText,
  },
  {
    titleKey: 'nav.chat',
    href: '/chat',
    icon: MessageSquare,
    roles: ['owner', 'admin'],
  },
];

/** Bottom sidebar items (Settings, Members) */
export const SIDEBAR_FOOTER_NAV: NavItemConfig[] = [
  {
    titleKey: 'nav.members',
    href: '/members',
    icon: Users,
    roles: ['owner', 'admin'],
  },
  {
    titleKey: 'nav.settings',
    href: '/settings',
    icon: Settings,
    roles: ['owner'],
  },
];
```

---

## 🔑 Rules

- **Translation keys:** `titleKey` dùng l10n key, không phải string hiển thị trực tiếp.
- **Roles (optional):** Nếu không set, mọi role đều thấy. Nếu set, chỉ role liệt kê mới thấy.
- **Thêm menu mới:** Chỉ cần thêm object vào array, không sửa UI component.

---

## 📚 Related
- [../components/03-navigation.md](../components/03-navigation.md) — Component render từ config này
- [../components/01-sidebar.md](../components/01-sidebar.md) — Sidebar sử dụng config

---

*Last Updated: 2026-02-11*
