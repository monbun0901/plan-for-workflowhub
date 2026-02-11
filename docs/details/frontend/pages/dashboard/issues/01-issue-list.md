# Issue List Page

**Status:** 🟢 Phase 1 (MVP)
**Route:** `/projects/:id/issues`
**Layout:** [Shell Layout](../../../layouts/01-shell-layout.md) (Tab bên trong Project Detail)

---

## 📐 Wireframe

```
┌──────────────────────────────────────┐
│  (Nằm trong Project Detail tab)      │
│                                      │
│  ┌─ Actions Bar ─────────────────┐   │
│  │ [Search...] [Severity▼] [+ New]│  │
│  └───────────────────────────────┘   │
│                                      │
│  ┌─ Table ───────────────────────┐   │
│  │ # │ Title    │ Severity │ Status│  │
│  │───┼──────────┼──────────┼──────│  │
│  │ 1 │ Bug #12  │ 🔴 Crit  │ Open │  │
│  │ 2 │ Bug #13  │ 🟡 Med   │ WIP  │  │
│  │ 3 │ Bug #14  │ 🟢 Low   │ Fixed│  │
│  └───────────────────────────────┘   │
│                                      │
│  [< 1 2 3 >]                         │
└──────────────────────────────────────┘
```

---

## ✅ Chức năng

| Feature | Status | Mô tả |
|---------|--------|--------|
| Issue table | 🟢 MVP | Cards (mobile) / Table (desktop) |
| Search by title | 🟢 MVP | Debounced |
| Filter by severity | 🟢 MVP | Critical/High/Medium/Low |
| **Filter by status** | 🟢 MVP | From `workflow_statuses` (target_type='issue') |
| Create Issue → page | 🟢 MVP | `/projects/:id/issues/new` |
| Severity badge | 🟢 MVP | Color-coded |
| Empty state | 🟢 MVP | "No issues found" + CTA |

---

## 🪝 Hooks

| Hook | Chức năng |
|------|----------|
| `useIssues(projectId)` | TanStack Query: list + filter |

## 📡 API Endpoints

| Method | Endpoint | Mô tả |
|--------|----------|--------|
| `GET` | `/:orgId/projects/:id/issues?severity=...&status=...` | Filtered list |
| `GET` | `/:orgId/lookups/workflow-statuses?target_type=issue` | Fetch issue status options |

---

*Last Updated: 2026-02-11*
