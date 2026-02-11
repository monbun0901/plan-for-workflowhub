# Project List Page

**Status:** 🟢 Phase 1 (MVP)
**Route:** `/projects`
**Layout:** [Shell Layout](../../../layouts/01-shell-layout.md)

---

## 📐 Wireframe

```
┌─ Sidebar ─┬──────────────────────────────────────┐
│            │  Header: Breadcrumbs + UserNav        │
│            ├──────────────────────────────────────┤
│            │                                      │
│            │  ┌─ Page Header ────────────────┐    │
│            │  │ "Projects"    [+ New Project] │    │
│            │  └──────────────────────────────┘    │
│            │                                      │
│            │  ┌─ Filters & Search ───────────┐    │
│            │  │ [Search...] [Status ▼] [Sort]│    │
│            │  └──────────────────────────────┘    │
│            │                                      │
│            │  ┌─ Grid (auto-fill) ───────────┐   │
│            │  │ ┌──────┐ ┌──────┐ ┌──────┐   │   │
│            │  │ │Card 1│ │Card 2│ │Card 3│   │   │
│            │  │ │status│ │status│ │status│   │   │
│            │  │ │ members│ members│ members  │   │
│            │  │ └──────┘ └──────┘ └──────┘   │   │
│            │  │                               │   │
│            │  │ ┌──────┐ ┌──────┐             │   │
│            │  │ │Card 4│ │Card 5│             │   │
│            │  │ └──────┘ └──────┘             │   │
│            │  └──────────────────────────────┘    │
│            │                                      │
│            │  [Pagination: < 1 2 3 ... 10 >]      │
└────────────┴──────────────────────────────────────┘
```

---

## ✅ Chức năng

| Feature | Status | Mô tả |
|---------|--------|--------|
| Project grid (responsive) | 🟢 MVP | Auto-fill cards |
| Search by name | 🟢 MVP | Debounced search |
| **Filter by status** | 🟢 MVP | From `workflow_statuses` (target_type='project') |
| Sort (newest, name) | 🟢 MVP | Sort dropdown |
| Create Project button | 🟢 MVP | Navigate → `/projects/new` |
| Pagination | 🟢 MVP | Server-side pagination |
| Empty State | 🟢 MVP | Illustration + CTA "Create first project" |
| Skeleton loading | 🟢 MVP | Grid skeleton matching layout |
| Bulk actions | 🟡 Scale | Multi-select + archive |

---

## 🧩 Components

| Component | Source |
|-----------|--------|
| `<ProjectCard />` | features/projects |
| `<SearchInput />` | shared |
| `<FilterDropdown />` | shared |
| `<EmptyState />` | shared |
| `<Pagination />` | shared |

## 🪝 Hooks

| Hook | Chức năng |
|------|----------|
| `useProjects()` | TanStack Query: list + filter + pagination |
| `useProjectFilters()` | URL-based filter state (searchParams) |

## 📡 API Endpoints

| Method | Endpoint | Mô tả |
|--------|----------|--------|
| `GET` | `/:orgId/projects?page=1&limit=20&status=...&search=...` | Filtered list |
| `GET` | `/:orgId/lookups/workflow-statuses?target_type=project` | Fetch project status options |

---

*Last Updated: 2026-02-11*
