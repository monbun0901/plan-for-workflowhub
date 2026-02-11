# Task List Page

**Status:** 🟢 Phase 1 (MVP)
**Route:** `/projects/:id/tasks`
**Layout:** [Shell Layout](../../../layouts/01-shell-layout.md) (Tab bên trong Project Detail)

---

## 📐 Wireframe

```
┌──────────────────────────────────────────────────────┐
│  (Nằm trong Project Detail tab)                      │
│                                                      │
│  ┌─ Actions Bar ───────────────────────────────┐    │
│  │ [Search...] [Assignees▼] [Assignments▼]     │    │
│  │ [Tags▼] [Category▼] [Status▼]  [⚏][☰] [+New]│    │
│  └─────────────────────────────────────────────┘    │
│                                                      │
│  ┌─ Grid View ─────────────────────────────────┐    │
│  │ ┌────────┐ ┌────────┐ ┌────────┐            │    │
│  │ │Task A  │ │Task B  │ │Task C  │            │    │
│  │ │👤User1 │ │👤User2 │ │—       │            │    │
│  │ │🔴High  │ │🟡Med   │ │🟢Low   │            │    │
│  │ └────────┘ └────────┘ └────────┘            │    │
│  └─────────────────────────────────────────────┘    │
│                                                      │
│  ┌─ Table View (alternate) ───────────────────┐     │
│  │ # │ Title    │ Assignee │ Category │ Prio  │     │
│  │───┼──────────┼──────────┼──────────┼───────│     │
│  │ 1 │ Task A   │ 👤 User1 │ Backend  │ 🔴    │     │
│  │ 2 │ Task B   │ 👤 User2 │ Frontend │ 🟡    │     │
│  └─────────────────────────────────────────────┘     │
│                                                      │
│  [< 1 2 3 >]                                         │
└──────────────────────────────────────────────────────┘
```

---

## ✅ Chức năng

| Feature | Status | Mô tả |
|---------|--------|--------|
| **View toggle (Grid/Table)** | 🟢 MVP | Toggle icons ⚏/☰, persist preference |
| Search by title | 🟢 MVP | Debounced |
| **Filter: Assignees** | 🟢 MVP | Multi-select, filter by assigned users |
| **Filter: Assignments** | 🟢 MVP | Multi-select, filter by task_assignments |
| **Filter: Tags** | 🟢 MVP | Multi-select tag filter |
| **Filter: Category** | 🟢 MVP | Single-select category |
| **Filter: Status** | 🟢 MVP | Single-select from `task_status` table |
| Create Task → `/projects/:id/tasks/new` | 🟢 MVP | Page-based |
| Inline status toggle | 🟢 MVP | Quick status change |
| Priority badge | 🟢 MVP | Color-coded badges |
| Drag & drop reorder | 🟡 Scale | Sortable |
| Kanban view (board) | 🟡 Scale | 3rd view mode option |
| Empty state | 🟢 MVP | "No tasks yet" + CTA |

---

## 🪝 Hooks

| Hook | Chức năng |
|------|----------|
| `useTasks(projectId, filters)` | TanStack Query: list + advanced filters |
| `useUpdateTaskStatus()` | Inline mutation |
| `useViewMode()` | Persist grid/table preference |

## 🗄️ Stores

| Store | Sử dụng |
|-------|---------|
| `useConfigStore` | Save view mode preference |

## 📡 API Endpoints

| Method | Endpoint | Mô tả |
|--------|----------|--------|
| `GET` | `/projects/:id/tasks?assignees=...&assignments=...&tags=...&category=...&status=...` | Advanced filtered list |
| `PATCH` | `/projects/:id/tasks/:taskId` | Quick status update |
| `GET` | `/task-statuses` | Fetch status options (from `task_status` table) |

## 🗂️ Database

**New Table Required:** `task_status`

| Column | Type | Description |
|--------|------|-------------|
| `id` | UUID | Primary key |
| `name` | VARCHAR | Status name (e.g., "Todo", "In Progress") |
| `slug` | VARCHAR | URL-safe slug |
| `color` | VARCHAR | Badge color (#hex) |
| `order` | INT | Display order |
| `is_default` | BOOLEAN | Default status |

---

*Last Updated: 2026-02-11*
