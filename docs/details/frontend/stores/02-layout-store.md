# Layout Store

**Version:** v1.0.0
**Status:** 🗄️ Zustand Store
**Path:** `src/stores/useLayoutStore.ts`

---

## 🎯 Scope

Quản lý UI layout state (sidebar, header, list view modes).

---

## 📦 State Schema

```typescript
interface LayoutState {
  // Sidebar
  sidebarState: 'expanded' | 'collapsed' | 'overlay';
  setSidebarState: (state: 'expanded' | 'collapsed' | 'overlay') => void;
  toggleSidebar: () => void;
  
  // Header Context
  headerContext: {
    showBreadcrumbs: boolean;
    showSearch: boolean;
    customActions?: ReactNode;
  };
  setHeaderContext: (context: Partial<LayoutState['headerContext']>) => void;
  
  // List View Modes (per page)
  listViewModes: Record<string, 'grid' | 'table'>; // { 'tasks': 'grid', 'projects': 'table' }
  setListViewMode: (pageKey: string, mode: 'grid' | 'table') => void;
  getListViewMode: (pageKey: string) => 'grid' | 'table'; // Default: 'grid'
}
```

---

## 🔧 Implementation

```typescript
/**
 * @follows senior-architect: Layout State Management
 * Persist sidebar + view modes vào localStorage
 */
import { create } from 'zustand';
import { persist } from 'zustand/middleware';

export const useLayoutStore = create<LayoutState>()(
  persist(
    (set, get) => ({
      // Sidebar State
      sidebarState: 'expanded',
      setSidebarState: (state) => set({ sidebarState: state }),
      toggleSidebar: () => {
        const current = get().sidebarState;
        const next = current === 'expanded' ? 'collapsed' : 'expanded';
        set({ sidebarState: next });
      },
      
      // Header Context
      headerContext: {
        showBreadcrumbs: true,
        showSearch: false,
      },
      setHeaderContext: (context) =>
        set((state) => ({
          headerContext: { ...state.headerContext, ...context },
        })),
      
      // List View Modes
      listViewModes: {}, // Empty default, will be populated on first use
      setListViewMode: (pageKey, mode) =>
        set((state) => ({
          listViewModes: { ...state.listViewModes, [pageKey]: mode },
        })),
      getListViewMode: (pageKey) => {
        const modes = get().listViewModes;
        return modes[pageKey] || 'grid'; // Default: grid
      },
    }),
    {
      name: 'layout-store', // localStorage key
      partialize: (state) => ({
        sidebarState: state.sidebarState,
        listViewModes: state.listViewModes, // Persist view modes
      }),
    }
  )
);
```

---

## 📚 Usage Examples

### List View Toggle Component
```typescript
/**
 * Generic ViewToggle component cho tất cả list pages
 */
import { useLayoutStore } from '@/stores/useLayoutStore';

function ViewToggle({ pageKey }: { pageKey: string }) {
  const viewMode = useLayoutStore((s) => s.getListViewMode(pageKey));
  const setViewMode = useLayoutStore((s) => s.setListViewMode);
  
  return (
    <div className="flex gap-1">
      <button
        onClick={() => setViewMode(pageKey, 'grid')}
        className={viewMode === 'grid' ? 'active' : ''}
      >
        <GridIcon />
      </button>
      <button
        onClick={() => setViewMode(pageKey, 'table')}
        className={viewMode === 'table' ? 'active' : ''}
      >
        <TableIcon />
      </button>
    </div>
  );
}

// Usage in TaskListPage
<ViewToggle pageKey="tasks" />
```

### Conditional Rendering by View Mode
```typescript
function TaskListPage() {
  const viewMode = useLayoutStore((s) => s.getListViewMode('tasks'));
  
  return (
    <>
      <ViewToggle pageKey="tasks" />
      {viewMode === 'grid' ? (
        <TaskGrid tasks={tasks} />
      ) : (
        <TaskTable tasks={tasks} />
      )}
    </>
  );
}
```

---

## 🎨 Design Rules

- **Persistence:** View mode được lưu per-page (key: 'tasks', 'projects', 'documents'...).
- **Default:** Mặc định là `'grid'` nếu chưa có preference.
- **Global Sidebar:** `sidebarState` persist globally, không phụ thuộc page.
- **Header Context:** Không persist, reset mỗi page change.

---

## 📚 Related
- [../components/01-sidebar.md](../components/01-sidebar.md) — Sidebar component
- [../components/02-header.md](../components/02-header.md) — Header component
- [../components/05-data-grid.md](../components/05-data-grid.md) — View toggle pattern

---

*Last Updated: 2026-02-11*
