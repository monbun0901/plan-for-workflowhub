# Edit Task Page

**Status:** 🟢 Phase 1 (MVP)
**Route:** `/projects/:id/tasks/:taskId/edit`
**Layout:** [CRUD Page Layout](../../../layouts/03-crud-page-layout.md)

---

## 📐 Wireframe

Giống `03-task-create.md` nhưng pre-filled với dữ liệu hiện tại.

```
┌─ Sidebar ─┬──────────────────────────────────┐
│            │  Header: Project > Tasks > Edit    │
│            ├──────────────────────────────────┤
│            │     max-w-2xl (centered)          │
│            │                                  │
│            │  ┌─ Page Header ──────────────┐  │
│            │  │ "Edit Task"        [Cancel]  │  │
│            │  └────────────────────────────┘  │
│            │                                  │
│            │  ┌─ Form Card (pre-filled) ───┐  │
│            │  │  Title:    [Existing title]  │  │
│            │  │  Priority: [High ▼       ]   │  │
│            │  │  Status:   [In Progress ▼]   │  │
│            │  │  Assignee: [👤 User1     ]   │  │
│            │  │  Due Date: [📅 2026-03-15]   │  │
│            │  │  Labels:   [🏷️ Bug] [🏷️ UI]  │  │
│            │  │  Desc:     [Existing...]     │  │
│            │  │                              │  │
│            │  │  [Cancel]       [Save]       │  │
│            │  └──────────────────────────────┘ │
└────────────┴──────────────────────────────────┘
```

---

## ✅ Chức năng

| Feature | Status | Mô tả |
|---------|--------|--------|
| Pre-fill form data | 🟢 MVP | Load existing task data |
| All fields editable | 🟢 MVP | Title, priority, status, assignee, due, labels, desc |
| Dirty form detection | 🟢 MVP | Warn on unsaved changes |
| Submit → Toast + redirect | 🟢 MVP | Non-blocking, redirect `/projects/:id/tasks/:taskId` |

---

## 🪝 Hooks

| Hook | Chức năng |
|------|----------|
| `useTask(projectId, taskId)` | Fetch existing data |
| `useEditTaskForm(task)` | Form state pre-filled |
| `useUpdateTask()` | Mutation + Toast + redirect |

## 📡 API Endpoints

| Method | Endpoint | Mô tả |
|--------|----------|--------|
| `GET` | `/projects/:id/tasks/:taskId` | Load current data |
| `PUT` | `/projects/:id/tasks/:taskId` | Update task |

---

*Last Updated: 2026-02-11*
