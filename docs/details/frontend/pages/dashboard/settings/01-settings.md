# Settings Page

**Status:** 🟢 Phase 1 (MVP)
**Route:** `/settings`
**Layout:** [Shell Layout](../../../layouts/01-shell-layout.md)

---

## 📐 Wireframe

```
┌─ Sidebar ─┬──────────────────────────────────────┐
│            │  Header: "Settings" + UserNav          │
│            ├──────────────────────────────────────┤
│            │                                      │
│            │  ┌─ Settings Tabs ──────────────┐    │
│            │  │ [Profile] [Organization] [App]│    │
│            │  └──────────────────────────────┘    │
│            │                                      │
│            │  ┌─ Profile Tab ────────────────┐    │
│            │  │  Avatar: 👤 [Upload]          │    │
│            │  │  Name:   [____________]       │    │
│            │  │  Email:  user@email (locked)  │    │
│            │  │  Timezone: [UTC+7 ▼]          │    │
│            │  │                               │    │
│            │  │  [Save Changes]               │    │
│            │  └──────────────────────────────┘    │
│            │                                      │
│            │  ┌─ Danger Zone ────────────────┐    │
│            │  │ ⚠ Change Password             │    │
│            │  │ 🔴 Delete Account             │    │
│            │  └──────────────────────────────┘    │
└────────────┴──────────────────────────────────────┘
```

---

## ✅ Chức năng

### Tab: Profile
| Feature | Status | Mô tả |
|---------|--------|--------|
| Update avatar | 🟢 MVP | Upload image |
| Update name | 🟢 MVP | Text input |
| Email (read-only) | 🟢 MVP | Locked field |
| Timezone selector | 🟢 MVP | Dropdown |
| Change password | 🟢 MVP | Old + New + Confirm |

### Tab: Organization (Owner/Admin only)
| Feature | Status | Mô tả |
|---------|--------|--------|
| Org name | 🟢 MVP | Editable |
| Org logo | 🟢 MVP | Upload |
| Delete organization | 🟢 MVP | Double-confirm |

### Tab: App Preferences
| Feature | Status | Mô tả |
|---------|--------|--------|
| Theme (Light/Dark/System) | 🟢 MVP | Radio group |
| Language (vi/en) | 🟢 MVP | Dropdown |
| Notification preferences | 🟡 Scale | Toggle per type |

---

## 🪝 Hooks

| Hook | Chức năng |
|------|----------|
| `useProfileForm()` | Form state + Zod |
| `useUpdateProfile()` | Mutation + Toast |
| `useOrgSettingsForm()` | Org-specific form |
| `useUpdateOrg()` | Mutation + Toast |

## 🗄️ Stores

| Store | Sử dụng |
|-------|---------|
| `useAuthStore` | Update user info after save |
| `useConfigStore` | Theme, Language changes |
| `useTenantStore` | Org info update |

## 📡 API Endpoints

| Method | Endpoint | Mô tả |
|--------|----------|--------|
| `PATCH` | `/auth/profile` | Update profile |
| `POST` | `/auth/change-password` | Change password |
| `PATCH` | `/organizations/:orgId` | Update org |
| `DELETE` | `/organizations/:orgId` | Delete org |

---

*Last Updated: 2026-02-11*
