# Create Project Page

**Status:** 🟢 Phase 1 (MVP)
**Route:** `/projects/new`
**Layout:** [CRUD Page Layout](../../../layouts/03-crud-page-layout.md)

---

## 📐 Wireframe

```
┌─ Sidebar ─┬──────────────────────────────────┐
│            │  Header: Breadcrumbs              │
│            ├──────────────────────────────────┤
│            │     max-w-2xl (centered)          │
│            │                                  │
│            │  ┌─ Page Header ──────────────┐  │
│            │  │ "Create New Project" [Cancel]│  │
│            │  └────────────────────────────┘  │
│            │                                  │
│            │  ┌─ Form Card ────────────────┐  │
│            │  │                             │  │
│            │  │  Name:     [____________]   │  │
│            │  │  Key:      [____] (auto)    │  │
│            │  │  Desc:     [____________]   │  │
│            │  │            [____________]   │  │
│            │  │  Status:   [Active ▼   ]    │  │
│            │  │                             │  │
│            │  │  ┌─────────────────────┐    │  │
│            │  │  │ [Cancel]    [Save]  │    │  │
│            │  │  └─────────────────────┘    │  │
│            │  └─────────────────────────────┘ │
└────────────┴──────────────────────────────────┘
```

---

## ✅ Chức năng

| Feature | Status | Mô tả |
|---------|--------|--------|
| Name field (required) | 🟢 MVP | Validation: min 3, max 100 chars |
| Project Key (auto-gen) | 🟢 MVP | Tự sinh từ name, có thể chỉnh |
| Description (optional) | 🟢 MVP | Textarea |
| Status selector | 🟢 MVP | Default: Active |
| Cancel → router.back() | 🟢 MVP | Quay về trang trước |
| Submit → Toast + redirect | 🟢 MVP | Non-blocking, redirect `/projects` |
| Template selection | 🟡 Scale | Chọn template dự án |

---

## 🧩 Components

| Component | Source |
|-----------|--------|
| `<ProjectForm />` | features/projects |
| `<FormField />` | shared/forms |
| `<FormActions />` | shared/forms |

## 🪝 Hooks

| Hook | Chức năng |
|------|----------|
| `useCreateProjectForm()` | Form state + Zod validation |
| `useCreateProject()` | Mutation + Toast + redirect |

## 📡 API Endpoints

| Method | Endpoint | Mô tả |
|--------|----------|--------|
| `POST` | `/projects` | Create new project |

---

*Last Updated: 2026-02-11*
