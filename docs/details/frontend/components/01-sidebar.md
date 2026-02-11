# Sidebar Component

**Version:** v1.0.0
**Status:** 🧩 Core Component
**Path:** `src/components/layout/Sidebar.tsx`, `src/components/layout/SidebarContent.tsx`

---

## 🔄 Triple-State Sidebar

Sidebar hoạt động theo 3 trạng thái, đồng bộ thông qua `useLayoutStore`.

| State | Breakpoint | Width | Hiển thị |
|-------|-----------|-------|----------|
| 🟢 Expand | `≥ 1024px` | `15rem` (240px) | Icon + Text label |
| 🟡 Collapse | `≥ 1024px` | `4rem` (64px) | Chỉ Icon |
| 🔴 Mobile Overlay | `< 1024px` | `85%` max `18.75rem` | Fullscreen Drawer |

---

## 🏗️ Component Structure

```
<Sidebar>
  ├── <SidebarHeader />        # Logo / Brand
  ├── <SidebarNav />           # Navigation items (data-driven)
  │   └── <NavItem />          # Từng menu item
  ├── <SidebarFooter />        # User info, Logout
  └── <SidebarToggle />        # Nút Expand/Collapse
```

---

## 📝 SidebarContent Spec

```tsx
interface SidebarContentProps {
  /** Trạng thái thu gọn (Desktop only) */
  collapsed?: boolean;
  /** Hiển thị đầy đủ (Mobile Drawer) */
  fullWidth?: boolean;
}

/**
 * @follows senior-architect: Data-driven Navigation
 * Nhận cấu hình từ constants, không hard-code menu
 */
export function SidebarContent({ collapsed = false, fullWidth = false }: SidebarContentProps) {
  const { t } = useTranslation();

  return (
    <div className="flex h-full flex-col">
      {/* Brand */}
      <SidebarHeader collapsed={collapsed} />

      {/* Navigation - Data-driven từ constants */}
      <SidebarNav 
        items={DASHBOARD_NAV} 
        isCollapsed={collapsed && !fullWidth} 
      />

      {/* Footer */}
      <SidebarFooter collapsed={collapsed} />
    </div>
  );
}
```

---

## 🎨 Styling Rules

- **Transition:** `transition-all duration-300 ease-in-out` cho hiệu ứng mượt.
- **Fixed position:** Sidebar cố định bên trái (`fixed left-0 top-0`).
- **Z-index:** Desktop `z-50`, Mobile Overlay `z-[100]`.
- **Backdrop:** Mobile overlay có `bg-background/80 backdrop-blur-sm`.
- **Đơn vị:** Sử dụng `rem`, không dùng `px`.

---

## 📚 Related
- [../layouts/01-shell-layout.md](../layouts/01-shell-layout.md) — Parent layout
- [03-navigation.md](03-navigation.md) — Data-driven nav config
- [../stores/02-layout-store.md](../stores/02-layout-store.md) — useLayoutStore
- [../constants/01-navigation-config.md](../constants/01-navigation-config.md) — Menu items

---

*Last Updated: 2026-02-11*
