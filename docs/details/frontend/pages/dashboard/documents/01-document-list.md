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
| **View toggle (Grid/Table)** | 🟡 Scale | Toggle icons ⚏/☰, persist preference |
| Document grid/table | 🟡 Scale | Card or table (icon + title + date) |
| Search by title | 🟡 Scale | Debounced |
| **Filter: Category** | 🟡 Scale | Single-select category filter |
| **Filter: Collaborators** | 🟡 Scale | Multi-select users who have access |
| **Filter: Tags** | 🟡 Scale | Multi-select tag filter |
| Create Document → page | 🟡 Scale | `/documents/new` |
| Delete document | 🟡 Scale | Confirm dialog |
| RAG-ready indexing | 🔴 Coming | Auto-index cho AI search |

---

## 🪝 Hooks

| Hook | Chức năng |
|------|----------|
| `useDocuments(projectId, filters)` | TanStack Query with advanced filters |
| `useViewMode()` | Persist grid/table preference |

## 🗄️ Stores

| Store | Sử dụng |
|-------|---------|
| `useConfigStore` | Save view mode preference |

## 📡 API Endpoints

| Method | Endpoint | Mô tả |
|--------|----------|--------|
| `GET` | `/projects/:id/documents?category=...&collaborators=...&tags=...` | Filtered list |
| `DELETE` | `/documents/:docId` | Delete document |

---

*Last Updated: 2026-02-11*
