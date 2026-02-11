# Edit Project Page

**Status:** 🟢 Phase 1 (MVP)
**Route:** `/projects/:id/edit`
**Layout:** [CRUD Page Layout](../../../layouts/03-crud-page-layout.md)

---

## 📐 Wireframe

Giống `02-project-create.md` nhưng pre-filled với dữ liệu hiện tại.

```
┌─ Sidebar ─┬──────────────────────────────────┐
│            │  Header: Projects > Name > Edit   │
│            ├──────────────────────────────────┤
│            │     max-w-2xl (centered)          │
│            │                                  │
│            │  ┌─ Page Header ──────────────┐  │
│            │  │ "Edit Project"     [Cancel] │  │
│            │  └────────────────────────────┘  │
│            │                                  │
│            │  ┌─ Form Card (pre-filled) ───┐  │
│            │  │  Name:     [Project Alpha]  │  │
│            │  │  Key:      [PA] (locked)    │  │
│            │  │  Desc:     [Existing desc]  │  │
│            │  │  Status:   [Active ▼   ]    │  │
│            │  │                             │  │
│            │  │  [Cancel]         [Save]    │  │
│            │  └─────────────────────────────┘ │
└────────────┴──────────────────────────────────┘
```

---

## ✅ Chức năng

| Feature | Status | Mô tả |
|---------|--------|--------|
| Pre-fill form data | 🟢 MVP | Load existing project data |
| Project Key (locked) | 🟢 MVP | Không cho phép sửa key |
| Submit → Toast + redirect | 🟢 MVP | Non-blocking, redirect `/projects/:id` |
| Dirty form detection | 🟢 MVP | Cảnh báo nếu rời trang chưa save |

---

## 🪝 Hooks

| Hook | Chức năng |
|------|----------|
| `useProject(id)` | Fetch existing data |
| `useEditProjectForm(project)` | Form state pre-filled |
| `useUpdateProject()` | Mutation + Toast + redirect |

## 📡 API Endpoints

| Method | Endpoint | Mô tả |
|--------|----------|--------|
| `GET` | `/projects/:id` | Load current data |
| `PUT` | `/projects/:id` | Update project |

---

*Last Updated: 2026-02-11*
