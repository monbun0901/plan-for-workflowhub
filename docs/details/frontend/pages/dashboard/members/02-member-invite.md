# Invite Member Page

**Status:** 🟢 Phase 1 (MVP)
**Route:** `/members/invite`
**Layout:** [CRUD Page Layout](../../../layouts/03-crud-page-layout.md)

---

## 📐 Wireframe

```
┌─ Sidebar ─┬──────────────────────────────────┐
│            │  Header: Members > Invite          │
│            ├──────────────────────────────────┤
│            │     max-w-2xl (centered)          │
│            │                                  │
│            │  ┌─ Page Header ──────────────┐  │
│            │  │ "Invite Member"    [Cancel] │  │
│            │  └────────────────────────────┘  │
│            │                                  │
│            │  ┌─ Form Card ────────────────┐  │
│            │  │                             │  │
│            │  │  Email:    [____________]   │  │
│            │  │  Role:     [Member ▼   ]    │  │
│            │  │  Message:  [____________]   │  │
│            │  │            (optional)       │  │
│            │  │                             │  │
│            │  │  [Cancel]         [Invite]  │  │
│            │  └─────────────────────────────┘ │
│            │                                  │
│            │  ┌─ Bulk Invite (Scale) ──────┐  │
│            │  │  [📎 Upload CSV]            │  │
│            │  └─────────────────────────────┘ │
└────────────┴──────────────────────────────────┘
```

---

## ✅ Chức năng

| Feature | Status | Mô tả |
|---------|--------|--------|
| Email input (required) | 🟢 MVP | Validation email format |
| Role selector | 🟢 MVP | Admin / Member (Owner cannot invite Owner) |
| Optional message | 🟢 MVP | Welcome message |
| Submit → Toast + redirect | 🟢 MVP | Non-blocking, redirect `/members` |
| Bulk invite (CSV upload) | 🟡 Scale | Import emails from file |
| Multi-email input | 🟡 Scale | Add multiple emails at once |

---

## 🪝 Hooks

| Hook | Chức năng |
|------|----------|
| `useInviteMemberForm()` | Form state + Zod validation |
| `useInviteMember()` | Mutation + Toast + redirect |

## 📡 API Endpoints

| Method | Endpoint | Mô tả |
|--------|----------|--------|
| `POST` | `/organizations/:orgId/invites` | Send invite |

---

*Last Updated: 2026-02-11*
