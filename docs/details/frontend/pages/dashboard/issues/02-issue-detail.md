# Issue Detail Page

**Status:** 🟢 Phase 1 (MVP)
**Route:** `/projects/:id/issues/:issueId`
**Layout:** [Shell Layout](../../../layouts/01-shell-layout.md)

---

## 📐 Wireframe

Layout tương tự Task Detail, với các trường dành riêng cho Issues.

```
┌─ Sidebar ─┬──────────────────────────────────────┐
│            │  Header: Project > Issues > #42       │
│            ├──────────────────────────────────────┤
│            │                                      │
│            │  ┌─ Main (max-w-3xl) ──── Sidebar ─┐│
│            │  │                    │              ││
│            │  │  [PROJ-42]         │  Status: ▼  ││
│            │  │  # Bug Title       │  Severity:▼ ││
│            │  │                    │  Assignee:  ││
│            │  │  Steps to reproduce│  👤 Dev     ││
│            │  │  1. ...            │  Reporter:  ││
│            │  │  2. ...            │  👤 QA      ││
│            │  │                    │  Labels:    ││
│            │  │  Expected:         │  🏷️ UI Bug  ││
│            │  │  Actual:           │              ││
│            │  │                    │              ││
│            │  │  💬 Comments       │              ││
│            │  │  [Add comment...] │              ││
│            │  └────────────────────┴──────────────┘│
└────────────┴──────────────────────────────────────┘
```

---

## ✅ Chức năng

| Feature | Status | Mô tả |
|---------|--------|--------|
| Issue info (title, description) | 🟢 MVP | Read/Edit |
| Steps to Reproduce | 🟢 MVP | Rich text |
| Expected vs Actual | 🟢 MVP | Comparison fields |
| Status changer | 🟢 MVP | Open/InProg/Fixed/Closed |
| Severity changer | 🟢 MVP | Critical/High/Med/Low |
| Assignee & Reporter | 🟢 MVP | Member pickers |
| Labels (tags) | 🟢 MVP | Multi-select tags |
| Comments | 🟢 MVP | Thread-style |
| Screenshots | 🟡 Scale | Image upload |
| Linked Tasks | 🟡 Scale | Relation to tasks |

---

## 🪝 Hooks

| Hook | Chức năng |
|------|----------|
| `useIssue(projectId, issueId)` | Fetch issue detail |
| `useUpdateIssue()` | Mutation for any field |
| `useComments(issueId)` | Fetch + add comments |

## 📡 API Endpoints

| Method | Endpoint | Mô tả |
|--------|----------|--------|
| `GET` | `/projects/:id/issues/:issueId` | Issue detail |
| `PATCH` | `/projects/:id/issues/:issueId` | Update |
| `GET` | `/issues/:issueId/comments` | Comments |
| `POST` | `/issues/:issueId/comments` | Add comment |

---

*Last Updated: 2026-02-11*
