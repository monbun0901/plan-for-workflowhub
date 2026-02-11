# Header Component

**Version:** v1.0.0
**Status:** 🧩 Core Component
**Path:** `src/components/layout/Header.tsx`

---

## 🎯 Contextual Header

Header thay đổi nội dung dựa trên ngữ cảnh (context) của trang hiện tại.

| Context | Left Side | Right Side |
|---------|-----------|------------|
| **Default** | Menu Toggle + Search | UserNav |
| **Inside Project** | Menu Toggle + Breadcrumbs | Quick Actions + UserNav |
| **Mobile** | Hamburger Menu + Page Title | UserNav (compact) |

---

## 🏗️ Component Structure

```
<Header>
  ├── <div className="left">
  │   ├── <SidebarToggleButton />  # Desktop: toggle collapse, Mobile: open drawer
  │   ├── <Breadcrumbs />          # Data-driven từ route
  │   └── <SearchTrigger />        # Command palette trigger (optional)
  │
  └── <div className="right">
      ├── <NotificationBell />
      └── <UserNav />              # Avatar + Dropdown menu
```

---

## 📝 Component Spec

```tsx
interface HeaderProps {
  /** Hiển thị breadcrumbs hoặc title */
  showBreadcrumbs?: boolean;
}

/**
 * @follows senior-architect: Contextual Header
 * Sticky header với backdrop blur
 */
export function Header({ showBreadcrumbs = true }: HeaderProps) {
  const { toggleSidebar, setOpen } = useLayoutStore();
  const { t } = useTranslation();

  return (
    <header className="sticky top-0 z-40 h-[3.5rem] w-full border-b bg-background/80 backdrop-blur-md">
      <div className="flex h-full items-center justify-between px-4">
        <div className="flex items-center gap-4">
          {/* Desktop Toggle */}
          <Button variant="ghost" size="icon" onClick={toggleSidebar} className="hidden lg:flex">
            <MenuIcon />
          </Button>
          {/* Mobile Hamburger */}
          <Button variant="ghost" size="icon" onClick={() => setOpen(true)} className="lg:hidden">
            <MenuIcon />
          </Button>
          {showBreadcrumbs && <Breadcrumbs />}
        </div>
        <UserNav />
      </div>
    </header>
  );
}
```

---

## 🎨 Styling Rules

- **Sticky:** `sticky top-0` để luôn hiện khi cuộn.
- **Backdrop glass:** `bg-background/80 backdrop-blur-md` cho hiệu ứng kính mờ.
- **Height:** `3.5rem` (56px equivalent).
- **Z-index:** `z-40` (dưới sidebar overlay `z-[100]`).

---

## 📚 Related
- [01-sidebar.md](01-sidebar.md) — Sidebar toggle integration
- [../layouts/01-shell-layout.md](../layouts/01-shell-layout.md) — Parent layout
- [../stores/02-layout-store.md](../stores/02-layout-store.md) — Layout state

---

*Last Updated: 2026-02-11*
