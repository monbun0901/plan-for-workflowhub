# ActionsBar Component

**Version:** v1.0.0
**Status:** 🧩 Reusable Component
**Path:** `src/components/shared/ActionsBar.tsx`

---

## 🎯 Purpose

Top action bar for list pages, containing search, filters, view toggle, and primary action buttons.

---

## 📦 Props Interface

```typescript
interface ActionsBarProps {
  searchPlaceholder?: string;        // Search input placeholder
  onSearchChange?: (value: string) => void;
  filterSlot?: ReactNode;            // Filter dropdowns area
  viewTogglePageKey?: string;        // Auto-render ViewToggle if provided
  actionSlot?: ReactNode;            // Primary actions (e.g., "+ New" button)
}
```

---

## 🧬 Component Structure

```tsx
/**
 * @follows senior-architect: Composition Pattern
 * Flexible action bar với slots
 */
import { MagnifyingGlassIcon } from '@heroicons/react/24/outline';
import { ViewToggle } from './ViewToggle';

export function ActionsBar({
  searchPlaceholder = 'Search...',
  onSearchChange,
  filterSlot,
  viewTogglePageKey,
  actionSlot,
}: ActionsBarProps) {
  return (
    <div className="flex flex-wrap items-center justify-between gap-3 border-b border-gray-200 bg-white px-6 py-3">
      {/* Left: Search + Filters */}
      <div className="flex flex-1 items-center gap-2">
        {onSearchChange && (
          <div className="relative">
            <MagnifyingGlassIcon className="absolute left-3 top-1/2 h-5 w-5 -translate-y-1/2 text-gray-400" />
            <input
              type="search"
              placeholder={searchPlaceholder}
              onChange={(e) => onSearchChange(e.target.value)}
              className="w-64 rounded-md border border-gray-300 py-2 pl-10 pr-3 text-sm focus:outline-none focus:ring-2 focus:ring-blue-500"
            />
          </div>
        )}
        
        {filterSlot && (
          <div className="flex items-center gap-2">{filterSlot}</div>
        )}
      </div>
      
      {/* Right: View Toggle + Actions */}
      <div className="flex items-center gap-2">
        {viewTogglePageKey && <ViewToggle pageKey={viewTogglePageKey} />}
        {actionSlot}
      </div>
    </div>
  );
}
```

---

## 📚 Usage Examples

### Simple: Search + Create Button
```tsx
<ActionsBar
  searchPlaceholder="Search tasks..."
  onSearchChange={(value) => setSearchQuery(value)}
  actionSlot={<Button href="/tasks/new">+ New Task</Button>}
/>
```

### With Filters + View Toggle
```tsx
<ActionsBar
  searchPlaceholder="Search projects..."
  onSearchChange={handleSearch}
  filterSlot={
    <>
      <DropdownFilter
        label="Status"
        options={statusOptions}
        value={statusFilter}
        onChange={setStatusFilter}
      />
      <DropdownFilter
        label="Category"
        options={categoryOptions}
        value={categoryFilter}
        onChange={setCategoryFilter}
      />
    </>
  }
  viewTogglePageKey="projects"
  actionSlot={<Button href="/projects/new">+ New Project</Button>}
/>
```

### Minimal (No Search)
```tsx
<ActionsBar
  filterSlot={<DropdownFilter label="Role" options={roles} />}
  actionSlot={<Button>+ Invite Member</Button>}
/>
```

---

## 🎨 Design Rules

- **Responsive:** Stack vertically trên mobile với `flex-wrap`.
- **Search Width:** Fixed `w-64` trên desktop, full-width trên mobile.
- **Spacing:** Consistent `gap-2` và `gap-3`.

---

*Last Updated: 2026-02-11*
