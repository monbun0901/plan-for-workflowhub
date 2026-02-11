# Create Issue Page

**Status:** 🟢 Phase 1 (MVP)
**Route:** `/projects/:id/issues/new`
**Layout:** [CRUD Page Layout](../../../layouts/03-crud-page-layout.md)

---

## 📐 Wireframe

```
┌─ Sidebar ─┬──────────────────────────────────┐
│            │  Header: Project > Issues > New    │
│            ├──────────────────────────────────┤
│            │     max-w-2xl (centered)          │
│            │                                  │
│            │  ┌─ Page Header ──────────────┐  │
│            │  │ "Report New Issue"  [Cancel] │  │
│            │  └────────────────────────────┘  │
│            │                                  │
│            │  ┌─ Form Card ────────────────┐  │
│            │  │                             │  │
│            │  │  Title:      [____________] │  │
│            │  │  Severity:   [Medium ▼    ] │  │
│            │  │  Assignee:   [Select... ▼ ] │  │
│            │  │  Labels:     [+ Add tag   ] │  │
│            │  │                             │  │
│            │  │  Description:               │  │
│            │  │  [________________________] │  │
│            │  │  [________________________] │  │
│            │  │                             │  │
│            │  │  Steps to Reproduce:        │  │
│            │  │  [________________________] │  │
│            │  │                             │  │
│            │  │  Expected Behavior:         │  │
│            │  │  [________________________] │  │
│            │  │                             │  │
│            │  │  Actual Behavior:           │  │
│            │  │  [________________________] │  │
│            │  │                             │  │
│            │  │  [Cancel]       [Submit]    │  │
│            │  └─────────────────────────────┘ │
└────────────┴──────────────────────────────────┘
```

---

## ✅ Chức năng

| Feature | Status | Mô tả |
|---------|--------|--------|
| Title (required) | 🟢 MVP | Min 5 chars |
| Severity selector | 🟢 MVP | Critical / High / Medium / Low |
| Assignee picker | 🟢 MVP | Dropdown org members |
| Labels / Tags | 🟢 MVP | Multi-select tags |
| Description | 🟢 MVP | Rich textarea |
| Steps to Reproduce | 🟢 MVP | Numbered textarea |
| Expected / Actual | 🟢 MVP | Two textareas |
| Submit → Toast + redirect | 🟢 MVP | Non-blocking, redirect `/projects/:id/issues` |
| Screenshot upload | 🟡 Scale | Drag & drop images |

---

## 🪝 Hooks

| Hook | Chức năng |
|------|----------|
| `useCreateIssueForm()` | Form state + Zod validation |
| `useCreateIssue()` | Mutation + Toast + redirect |
| `useMembers(orgId)` | Load members for assignee picker |

## 📡 API Endpoints

| Method | Endpoint | Mô tả |
|--------|----------|--------|
| `POST` | `/projects/:id/issues` | Create new issue |

---

*Last Updated: 2026-02-11*
