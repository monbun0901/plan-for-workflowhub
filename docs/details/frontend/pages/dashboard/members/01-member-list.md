# Members Page

**Status:** 🟢 Phase 1 (MVP)
**Route:** `/members`
**Layout:** [Shell Layout](../../../layouts/01-shell-layout.md)

---

## 📐 Wireframe

```
┌─ Sidebar ─┬──────────────────────────────────────┐
│            │  Header: "Members" + UserNav           │
│            ├──────────────────────────────────────┤
│            │                                      │
│            │  ┌─ Page Header ────────────────┐    │
│            │  │ "Members"     [+ Invite]      │    │
│            │  └──────────────────────────────┘    │
│            │                                      │
│            │  ┌─ Search & Filter ────────────┐    │
│            │  │ [Search...]   [Role ▼]        │    │
│            │  └──────────────────────────────┘    │
│            │                                      │
│            │  ┌─ Table ──────────────────────┐    │
│            │  │ Avatar│ Name    │ Role  │ ⋯  │    │
│            │  │───────┼─────────┼───────┼────│    │
│            │  │ 👤    │ Admin1  │ Owner │ ⋯  │    │
│            │  │ 👤    │ Dev1    │ Admin │ ⋯  │    │
│            │  │ 👤    │ Dev2    │ Member│ ⋯  │    │
│            │  └──────────────────────────────┘    │
│            │                                      │
│            │  ┌─ Pending Invites ────────────┐    │
│            │  │ email@... │ Pending │ [Resend]│    │
│            │  └──────────────────────────────┘    │
└────────────┴──────────────────────────────────────┘
```

---

## ✅ Chức năng

| Feature | Status | Mô tả |
|---------|--------|--------|
| Member table | 🟢 MVP | Avatar, name, email, role |
| Search by name/email | 🟢 MVP | Debounced |
| Filter by role | 🟢 MVP | Owner/Admin/Member |
| Invite member (email) | 🟢 MVP | → Page `/members/invite` |
| Change role | 🟢 MVP | Dropdown (Owner/Admin only) |
| Remove member | 🟢 MVP | Confirm dialog |
| Pending invites section | 🟢 MVP | List of outstanding invites |
| Resend invite | 🟢 MVP | Button per invite |
| Bulk invite (CSV) | 🟡 Scale | Import from file |

---

## 🪝 Hooks

| Hook | Chức năng |
|------|----------|
| `useMembers(orgId)` | List members |
| `useUpdateMemberRole()` | Change role mutation |
| `useRemoveMember()` | Remove + toast |
| `useInviteMember()` | Invite mutation |

## 📡 API Endpoints

| Method | Endpoint | Mô tả |
|--------|----------|--------|
| `GET` | `/organizations/:orgId/members` | List members |
| `PATCH` | `/organizations/:orgId/members/:id` | Update role |
| `DELETE` | `/organizations/:orgId/members/:id` | Remove |
| `POST` | `/organizations/:orgId/invites` | Invite |

---

*Last Updated: 2026-02-11*
