# Task Detail Page

**Status:** 🟢 Phase 1 (MVP)
**Route:** `/projects/:id/tasks/:taskId`
**Layout:** [Shell Layout](../../../layouts/01-shell-layout.md)

---

## 📐 Wireframe

```
┌─ Sidebar ─┬──────────────────────────────────────┐
│            │  Header: Project > Tasks > Task Title │
│            ├──────────────────────────────────────┤
│            │                                      │
│            │  ┌─ Main (max-w-3xl) ──── Sidebar ─┐│
│            │  │                    │              ││
│            │  │  [Task-KEY]        │  Status: ▼  ││
│            │  │  # Task Title      │  Priority:▼ ││
│            │  │                    │  Assignee:  ││
│            │  │  Description:      │  👤 User    ││
│            │  │  Long text here... │  Due: Date  ││
│            │  │                    │  Labels:    ││
│            │  │  ──────────────    │  🏷️ Bug     ││
│            │  │  💬 Comments       │              ││
│            │  │  ┌─────────────┐  │              ││
│            │  │  │ Comment 1   │  │              ││
│            │  │  │ Comment 2   │  │              ││
│            │  │  └─────────────┘  │              ││
│            │  │  [Add comment...] │              ││
│            │  │                    │              ││
│            │  └────────────────────┴──────────────┘│
└────────────┴──────────────────────────────────────┘
```

---

## ✅ Chức năng

| Feature | Status | Mô tả |
|---------|--------|--------|
| Task info (title, description) | 🟢 MVP | Read/Edit inline |
| Status changer | 🟢 MVP | Dropdown select |
| Priority changer | 🟢 MVP | Dropdown select |
| Assignee picker | 🟢 MVP | Member dropdown |
| Due date picker | 🟢 MVP | Date picker |
| Labels (tags) | 🟢 MVP | Multi-select tags |
| Comments section | 🟢 MVP | Thread-style |
| Add comment | 🟢 MVP | Textarea + submit |
| Edit task → inline | 🟢 MVP | Click-to-edit fields |
| File attachments | 🟡 Scale | Upload files |
| Time tracking | 🟡 Scale | Log hours |
| Sub-tasks | 🟡 Scale | Nested checklist |

---

## 🪝 Hooks

| Hook | Chức năng |
|------|----------|
| `useTask(projectId, taskId)` | Fetch task detail |
| `useUpdateTask()` | Mutation: status, priority, assignee |
| `useComments(taskId)` | Fetch + add comments |

## 📡 API Endpoints

| Method | Endpoint | Mô tả |
|--------|----------|--------|
| `GET` | `/projects/:id/tasks/:taskId` | Task detail |
| `PATCH` | `/projects/:id/tasks/:taskId` | Update any field |
| `GET` | `/tasks/:taskId/comments` | List comments |
| `POST` | `/tasks/:taskId/comments` | Add comment |

---

*Last Updated: 2026-02-11*
