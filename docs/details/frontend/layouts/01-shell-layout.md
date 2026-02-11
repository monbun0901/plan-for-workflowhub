# Shell Layout (Dashboard)

**Version:** v1.0.0
**Status:** 🏛️ Core Layout
**Path:** `src/app/(dashboard)/layout.tsx`

---

## 🏛️ Adaptive Shell Architecture

Cấu trúc Shell phải đạt cấp độ **"Adaptive"** (Thích nghi), không chỉ là Responsive đơn thuần.

### Thành phần chính
| Zone | Mô tả | Component |
|------|--------|-----------|
| **Sidebar** | Triple-State (Expand / Collapse / Mobile Overlay) | `<Sidebar />` |
| **Header** | Sticky, Contextual, chứa Breadcrumbs & UserNav | `<Header />` |
| **Main** | Content area, fluid, `max-w-screen-2xl` | `<main>` |

### Layout Flow
```
┌──────────┬──────────────────────────────┐
│          │  Header (sticky, z-40)       │
│ Sidebar  ├──────────────────────────────┤
│ (fixed)  │                              │
│          │  Main Content (flex-1)       │
│          │  └── max-w-screen-2xl        │
│          │                              │
└──────────┴──────────────────────────────┘
```

---

## 🏗️ Boilerplate Code

```tsx
/**
 * @follows senior-architect: Adaptive Shell Pattern
 * Quản lý Sidebar và Header theo trạng thái linh động
 */
export default function DashboardLayout({ children }) {
  const { isCollapsed, isOpen, toggleSidebar, setOpen } = useLayoutStore();

  return (
    <div className="relative flex min-h-screen bg-background text-foreground">
      {/* 1. MOBILE OVERLAY (Drawer Fullscreen) */}
      <div 
        className={cn(
          "fixed inset-0 z-[100] bg-background/80 backdrop-blur-sm lg:hidden",
          isOpen ? "block" : "hidden"
        )}
        onClick={() => setOpen(false)}
      >
        <aside 
          className="h-full w-[85%] max-w-[18.75rem] border-r bg-card p-6 shadow-xl"
          onClick={(e) => e.stopPropagation()}
        >
          <SidebarContent fullWidth />
        </aside>
      </div>

      {/* 2. DESKTOP SIDEBAR (Expand/Collapse) */}
      <aside 
        className={cn(
          "fixed left-0 top-0 z-50 h-screen border-r bg-card transition-all duration-300 ease-in-out hidden lg:block",
          isCollapsed ? "w-[4rem]" : "w-[15rem]"
        )}
      >
        <SidebarContent collapsed={isCollapsed} />
      </aside>

      {/* 3. MAIN WRAPPER */}
      <div 
        className={cn(
          "flex flex-1 flex-col transition-all duration-300",
          "lg:ml-[15rem]", 
          isCollapsed && "lg:ml-[4rem]"
        )}
      >
        <header className="sticky top-0 z-40 h-[3.5rem] w-full border-b bg-background/80 backdrop-blur-md">
          <div className="flex h-full items-center justify-between px-4">
            <div className="flex items-center gap-4">
              {/* Desktop Toggle */}
              <Button variant="ghost" size="icon" onClick={toggleSidebar} className="hidden lg:flex">
                <MenuIcon className="h-4 w-4" />
              </Button>
              {/* Mobile Menu */}
              <Button variant="ghost" size="icon" onClick={() => setOpen(true)} className="lg:hidden">
                <MenuIcon className="h-4 w-4" />
              </Button>
              <Breadcrumbs />
            </div>
            <UserNav />
          </div>
        </header>

        <main className="flex-1 p-4 md:p-6 lg:p-8">
          <div className="mx-auto max-w-screen-2xl">
            {children}
          </div>
        </main>
      </div>
    </div>
  );
}
```

---

## 📐 Responsive Breakpoints

| Breakpoint | Width | Sidebar State |
|------------|-------|---------------|
| Mobile | `< 1024px` | Hidden → Overlay Drawer |
| Desktop | `≥ 1024px` | Expand (`15rem`) hoặc Collapse (`4rem`) |

---

## 📚 Related
- [../components/01-sidebar.md](../components/01-sidebar.md) — Sidebar component spec
- [../components/02-header.md](../components/02-header.md) — Header component spec
- [../stores/02-layout-store.md](../stores/02-layout-store.md) — Layout state management

---

*Last Updated: 2026-02-11*
