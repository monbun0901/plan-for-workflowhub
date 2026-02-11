# Create Task Page

**Status:** 🟢 Phase 1 (MVP)
**Route:** `/projects/:id/tasks/new`
**Layout:** [CRUD Page Layout](../../../layouts/03-crud-page-layout.md)

---

## 📐 Wireframe

```
┌─ Sidebar ─┬──────────────────────────────────┐
│            │  Header: Project > Tasks > New     │
│            ├──────────────────────────────────┤
│            │     max-w-2xl (centered)          │
│            │                                  │
│            │  ┌─ Page Header ──────────────┐  │
│            │  │ "Create New Task"  [Cancel]  │  │
│            │  └────────────────────────────┘  │
│            │                                  │
│            │  ┌─ Form Card ────────────────┐  │
│            │  │                             │  │
│            │  │  Title:      [____________] │  │
│            │  │  Priority:   [Medium ▼    ] │  │
│            │  │  Status:     [Todo ▼      ] │  │
│            │  │  Assignee:   [Select... ▼ ] │  │
│            │  │  Due Date:   [📅 Pick...  ] │  │
│            │  │  Labels:     [+ Add tag   ] │  │
│            │  │                             │  │
│            │  │  Description:               │  │
│            │  │  [________________________] │  │
│            │  │  [________________________] │  │
│            │  │                             │  │
│            │  │  [Cancel]       [Create]    │  │
│            │  └─────────────────────────────┘ │
└────────────┴──────────────────────────────────┘
```

---

## ✅ Chức năng

| Feature | Status | Mô tả |
|---------|--------|--------|
| Title (required) | 🟢 MVP | Min 3 chars |
| Priority selector | 🟢 MVP | Low / Medium / High / Critical |
| Status selector | 🟢 MVP | Todo / In Progress (default: Todo) |
| Assignee picker | 🟢 MVP | Dropdown org members |
| Due date picker | 🟢 MVP | Calendar picker |
| Labels / Tags | 🟢 MVP | Multi-select tags |
| Description | 🟢 MVP | Rich textarea |
| Submit → Toast + redirect | 🟢 MVP | Non-blocking, redirect `/projects/:id/tasks` |
| Sub-tasks (checklist) | 🟡 Scale | Add nested items |
| Estimated hours | 🟡 Scale | Number input |

---

## 🪝 Hooks

| Hook | Chức năng |
|------|----------|
| `useCreateTaskForm()` | Form state + Zod validation |
| `useCreateTask()` | Mutation + Toast + redirect |
| `useMembers(orgId)` | Load members for assignee |

## 📡 API Endpoints

| Method | Endpoint | Mô tả |
|--------|----------|--------|
| `POST` | `/projects/:id/tasks` | Create new task |

---

*Last Updated: 2026-02-11*
