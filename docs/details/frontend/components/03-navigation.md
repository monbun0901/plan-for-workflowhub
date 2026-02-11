# Data-driven Navigation

**Version:** v1.0.0
**Status:** 🧩 Navigation Pattern
**Path:** `src/components/layout/SidebarNav.tsx`, `src/components/layout/NavItem.tsx`

---

## 🛠️ SoC Principle

Navigation items **không bao giờ hard-code** trong UI component. Component chỉ đóng vai trò **Renderer**, nhận dữ liệu từ config constants.

```
constants/navigation.ts  →  SidebarNav (Renderer)  →  NavItem (UI)
```

---

## 📝 NavItem Interface

```typescript
// src/types/navigation.ts
export interface NavItemConfig {
  /** Translation key cho label */
  titleKey: string;
  /** Route path */
  href: string;
  /** Lucide icon component */
  icon: LucideIcon;
  /** Roles được phép truy cập (optional, default: all) */
  roles?: string[];
  /** Sub-menu items (optional) */
  children?: NavItemConfig[];
  /** Badge count (optional, ví dụ: số notification) */
  badge?: number;
}
```

---

## 🏗️ SidebarNav Component

```tsx
/**
 * @follows senior-architect: Data-driven Navigation (SoC)
 * Chỉ render, không chứa logic định nghĩa menu
 */
export function SidebarNav({ items, isCollapsed }: SidebarNavProps) {
  const pathname = usePathname();
  const { t } = useTranslation();

  return (
    <nav className="flex-1 space-y-1 overflow-y-auto py-4">
      {items.map((item) => (
        <NavItem
          key={item.href}
          icon={item.icon}
          label={t(item.titleKey)}
          href={item.href}
          isActive={pathname.startsWith(item.href)}
          isCollapsed={isCollapsed}
          badge={item.badge}
        />
      ))}
    </nav>
  );
}
```

---

## 🎨 NavItem States

| State | Style |
|-------|-------|
| Default | `text-muted-foreground hover:bg-accent` |
| Active | `bg-accent text-accent-foreground font-medium` |
| Collapsed | Chỉ icon, tooltip hiện label |
| Disabled | `opacity-50 pointer-events-none` |

---

## 📚 Related
- [01-sidebar.md](01-sidebar.md) — Sidebar sử dụng SidebarNav
- [../constants/01-navigation-config.md](../constants/01-navigation-config.md) — Config data source

---

*Last Updated: 2026-02-11*
