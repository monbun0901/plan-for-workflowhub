# Data Grid & List Patterns

**Version:** v1.0.0
**Status:** 🧩 Display Pattern
**Path:** `src/components/shared/DataGrid.tsx`, `src/components/features/*/`

---

## 🧬 Fluid Grid Pattern (Auto-fill)

Grid tự co giãn theo viewport, không fix cứng số cột.

### Responsive Grid
```tsx
/**
 * @follows senior-architect: Fluid & Content-Driven Layout
 * Tự điều chỉnh số cột theo viewport
 */
<div className="grid grid-cols-1 gap-4 sm:grid-cols-2 lg:grid-cols-3 xl:grid-cols-4">
  {items.map(item => <Card key={item.id} {...item} />)}
</div>
```

### Content-Centric Container
Dùng cho các trang chi tiết (Document, Issue detail).
```tsx
<article className="mx-auto max-w-3xl space-y-6 px-4">
  <h1>{title}</h1>
  <div className="prose dark:prose-invert">
    {content}
  </div>
</article>
```

---

## 🔄 View Mode Toggle

Tất cả list pages phải hỗ trợ 2 view modes: **Grid** và **Table**.

### Toggle UI
```tsx
/**
 * @follows senior-architect: User Preference Persistence
 * View mode được lưu vào LocalStorage hoặc ConfigStore
 */
<div className="flex items-center gap-2">
  <button 
    onClick={() => setViewMode('grid')}
    className={viewMode === 'grid' ? 'active' : ''}
  >
    <GridIcon />
  </button>
  <button 
    onClick={() => setViewMode('table')}
    className={viewMode === 'table' ? 'active' : ''}
  >
    <TableIcon />
  </button>
</div>
```

### Conditional Rendering
```tsx
{viewMode === 'grid' ? (
  <div className="grid grid-cols-1 gap-4 sm:grid-cols-2 lg:grid-cols-3">
    {items.map(item => <Card key={item.id} {...item} />)}
  </div>
) : (
  <DataTable columns={columns} data={items} />
)}
```

---

## 🔍 Advanced Filter Bar Pattern

Filter bar được tùy biến theo từng page, không hard-code.

### Generic Filter Config
```tsx
/**
 * @follows senior-architect: Configuration-Driven UI
 * Filter options được define trong constants
 */
interface FilterConfig {
  key: string;
  label: string;
  type: 'select' | 'multiselect' | 'search';
  options?: Array<{ value: string; label: string }>;
}

// Example: tasks page filters
const TASK_FILTERS: FilterConfig[] = [
  { key: 'assignees', label: 'Assignees', type: 'multiselect' },
  { key: 'tags', label: 'Tags', type: 'multiselect' },
  { key: 'category', label: 'Category', type: 'select' },
  { key: 'status', label: 'Status', type: 'select' }, // ← workflow_statuses (target_type='task')
];

// Example: documents page filters
const DOC_FILTERS: FilterConfig[] = [
  { key: 'category', label: 'Category', type: 'select' },
  { key: 'collaborators', label: 'Collaborators', type: 'multiselect' },
  { key: 'tags', label: 'Tags', type: 'multiselect' },
];
```

### Reusable FilterBar Component
```tsx
/**
 * Generic FilterBar nhận config từ page
 */
<FilterBar
  filters={TASK_FILTERS}
  values={filterValues}
  onChange={setFilterValues}
/>
```

---

## 📊 Table Pattern

Dùng cho danh sách dữ liệu dạng bảng (Issues, Members).

### Responsive Strategy
| Breakpoint | Hiển thị |
|------------|---------|
| Mobile | Card list (stack vertical) |
| Tablet | Table với cột ẩn (hidden columns) |
| Desktop | Full table |

### Empty State
```tsx
/**
 * Luôn render Empty State khi không có data
 */
{items.length === 0 ? (
  <EmptyState
    icon={FolderOpenIcon}
    titleKey="project.empty_title"
    descriptionKey="project.empty_description"
    actionKey="project.create_first"
    onAction={() => router.push('/projects/new')}
  />
) : (
  <DataTable columns={columns} data={items} />
)}
```

---

## 🎨 Design Rules

- **View Toggle:** Mặc định Grid trên mobile, Table trên desktop.
- **Filter Persistence:** Lưu filter state vào URL query params.
- **Fluid:** Sử dụng `grid-cols-1` làm base, tăng dần theo breakpoint.
- **Gap:** Dùng `gap-4` (consistent spacing token).
- **Empty State:** Bắt buộc cho mọi danh sách.
- **Loading:** Skeleton grid cùng layout với data grid.

---

## 📚 Related
- [../layouts/01-shell-layout.md](../layouts/01-shell-layout.md) — Parent layout
- [../patterns/02-resilient-ui.md](../patterns/02-resilient-ui.md) — Resilient UI checklist
- [../constants/01-navigation-config.md](../constants/01-navigation-config.md) — Config-driven pattern

---

*Last Updated: 2026-02-11*
