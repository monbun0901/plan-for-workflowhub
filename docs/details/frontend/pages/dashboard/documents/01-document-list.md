# Document List Page

**Status:** 🟡 Scale-up
**Route:** `/projects/:id/documents`
**Layout:** [Shell Layout](../../../layouts/01-shell-layout.md) (Tab bên trong Project Detail)

---

## 📐 Wireframe

```
┌────────────────────────────────────────────┐
│  (Nằm trong Project Detail tab)            │
│                                            │
│  ┌─ Actions Bar ──────────────────────┐   │
│  │ [Search...] [Category▼] [Collabora-│   │
│  │ tors▼] [Tags▼]     [⚏][☰]  [+ New] │   │
│  └────────────────────────────────────┘   │
│                                            │
│  ┌─ Grid View ────────────────────────┐   │
│  │ ┌──────┐ ┌──────┐ ┌──────┐         │   │
│  │ │📄    │ │📄    │ │📄    │         │   │
│  │ │Doc 1 │ │Doc 2 │ │Doc 3 │         │   │
│  │ │2d ago│ │5d ago│ │1w ago│         │   │
│  │ └──────┘ └──────┘ └──────┘         │   │
│  └────────────────────────────────────┘   │
│                                            │
│  ┌─ Table View (alternate) ───────────┐   │
│  │ Name  │ Category │Collaborators│Date│   │
│  │───────┼──────────┼─────────────┼───│   │
│  │ Doc 1 │ Spec     │ 👤×2        │2d │   │
│  │ Doc 2 │ Design   │ 👤×3        │5d │   │
│  └────────────────────────────────────┘   │
└────────────────────────────────────────────┘
```

---

## ✅ Chức năng

| Feature | Status | Mô tả |
|---------|--------|--------|
| **View toggle (Grid/Table)** | 🟢 MVP | See [Data Grid Component](../../../components/05-data-grid.md) |
| Search by title | 🟢 MVP | Debounced |
| **Filter: Category** | 🟢 MVP | Single-select category |
| **Filter: Collaborators** | 🟢 MVP | Multi-select users who edited |
| **Filter: Tags** | 🟢 MVP | Multi-select tag filter |
| **Filter: Status** | 🟢 MVP | From `workflow_statuses` (target_type='document') |
| Create Document → page | 🟢 MVP | `/projects/:id/documents/new` |
| Empty state | 🟢 MVP | "No documents yet" + CTA |

---

## 🪝 Hooks

| Hook | Chức năng |
|------|----------|
| `useDocuments(projectId, filters)` | TanStack Query: list + advanced filters |
| `useViewMode()` | Persist grid/table preference |

## 🗄️ Stores

| Store | Sử dụng |
|-------|---------|
| `useConfigStore` | Save view mode preference |

## 📡 API Endpoints

| Method | Endpoint | Mô tả |
|--------|----------|--------|
| `GET` | `/:orgId/projects/:id/documents?category=...&collaborators=...&tags=...&status=...` | Advanced filtered list |
| `GET` | `/:orgId/lookups/workflow-statuses?target_type=document` | Fetch document status options |
| `DELETE` | `/documents/:docId` | Delete document |

---

*Last Updated: 2026-02-11*
