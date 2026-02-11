# Project Detail Page

**Status:** 🟢 Phase 1 (MVP)
**Route:** `/projects/:id`
**Layout:** [Shell Layout](../../../layouts/01-shell-layout.md)

---

## 📐 Wireframe

```
┌─ Sidebar ─┬──────────────────────────────────────┐
│            │  Header: Projects > Project Name      │
│            ├──────────────────────────────────────┤
│            │                                      │
│            │  ┌─ Project Header ──────────────┐   │
│            │  │ 📁 Project Name    [Edit] [⋯] │   │
│            │  │ Description text...            │   │
│            │  │ Status: ⬤ Active   Members: 5  │   │
│            │  └────────────────────────────────┘   │
│            │                                      │
│            │  ┌─ Tab Navigation ──────────────┐   │
│            │  │ [Tasks] [Issues] [Documents]  │   │
│            │  └────────────────────────────────┘   │
│            │                                      │
│            │  ┌─ Tab Content ─────────────────┐   │
│            │  │                                │   │
│            │  │  (Renders selected tab page)   │   │
│            │  │                                │   │
│            │  │  Task 1  │ Assignee │ Status   │   │
│            │  │  Task 2  │ Assignee │ Status   │   │
│            │  │  Task 3  │ Assignee │ Status   │   │
│            │  │                                │   │
│            │  └────────────────────────────────┘   │
└────────────┴──────────────────────────────────────┘
```

---

## ✅ Chức năng

| Feature | Status | Mô tả |
|---------|--------|--------|
| Project info header | 🟢 MVP | Name, description, status, members |
| Edit button → `/projects/:id/edit` | 🟢 MVP | Page-based CRUD |
| Tab: Tasks | 🟢 MVP | Inline task list |
| Tab: Issues | 🟢 MVP | Inline issue list |
| Tab: Documents | 🟡 Scale | Document list |
| More menu (Archive, Delete) | 🟢 MVP | Dropdown actions |
| Activity timeline | 🟡 Scale | Changelog sidebar |
| Kanban Board view | 🟡 Scale | Tasks as board |

---

## 🧩 Components

| Component | Source |
|-----------|--------|
| `<ProjectHeader />` | features/projects |
| `<TabNavigation />` | shared |
| `<TaskList />` | features/tasks |
| `<IssueList />` | features/issues |
| `<DropdownMenu />` | shadcn/ui |

## 🪝 Hooks

| Hook | Chức năng |
|------|----------|
| `useProject(id)` | Fetch project detail |
| `useDeleteProject()` | Mutation + confirm dialog |

## 📡 API Endpoints

| Method | Endpoint | Mô tả |
|--------|----------|--------|
| `GET` | `/projects/:id` | Project detail |
| `DELETE` | `/projects/:id` | Delete project |

---

*Last Updated: 2026-02-11*
