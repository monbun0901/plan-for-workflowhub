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
| **Filter: Assignees** | 🟢 MVP | Multi-select, current assigned users (from `task_assignees`) |
| **Filter: Tags** | 🟢 MVP | Multi-select tag filter |
| **Filter: Category** | 🟢 MVP | Single-select category |
| **Filter: Status** | 🟢 MVP | Single-select from `workflow_statuses` (target_type='task') |
| Create Task → `/projects/:id/tasks/new` | 🟢 MVP | Page-based |
| Inline status toggle | 🟢 MVP | Quick status change |
| Priority badge | 🟢 MVP | Color-coded badges |
| Drag & drop reorder | 🟡 Scale | Sortable |
| Kanban view (board) | 🟡 Scale | 3rd view mode option |
| View assignment history | 🟡 Scale | Show audit trail from `task_assignments` table |
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
| `GET` | `/:orgId/projects/:id/tasks?assignees=...&tags=...&category=...&status=...` | Advanced filtered list |
| `PATCH` | `/:orgId/projects/:id/tasks/:taskId` | Quick status update |
| `GET` | `/:orgId/lookups/workflow-statuses?target_type=task` | Fetch task status options |

## 🗂️ Database Tables

**Referenced Tables:**
- `task_assignees` - Current assignees (many-to-many, used for Assignees filter)
- `task_assignments` - Assignment history audit log (future: assignment history feature)
- `workflow_statuses` - Unified status table (filtered by target_type='task')

---

*Last Updated: 2026-02-11*
