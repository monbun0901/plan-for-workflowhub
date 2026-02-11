# Dashboard Overview Page

**Status:** 🟢 Phase 1 (MVP)
**Route:** `/dashboard`
**Layout:** [Shell Layout](../../layouts/01-shell-layout.md)

---

## 📐 Wireframe

```
┌─ Sidebar ─┬──────────────────────────────────┐
│            │  Header: "Dashboard" + UserNav    │
│  Overview  ├──────────────────────────────────┤
│  Projects  │                                  │
│  Tasks     │  ┌─ Stats Cards ──────────────┐  │
│  Issues    │  │ Projects │ Tasks │ Issues   │  │
│  Docs      │  │    12    │  45   │   8      │  │
│            │  └────────────────────────────┘  │
│            │                                  │
│            │  ┌─ Recent Projects ───────────┐ │
│            │  │ [Card] [Card] [Card]        │ │
│            │  └─────────────────────────────┘ │
│            │                                  │
│            │  ┌─ My Tasks ─────────────────┐  │
│            │  │ Task 1         ⬤ In Progress│  │
│            │  │ Task 2         ⬤ Review     │  │
│            │  │ Task 3         ⬤ Todo       │  │
│            │  └─────────────────────────────┘ │
│            │                                  │
│  Settings  │  ┌─ Activity Feed ────────────┐  │
│  Members   │  │ User A created Project X    │  │
│            │  │ User B closed Issue #42     │  │
└────────────┴──┴─────────────────────────────┘─┘
```

---

## ✅ Chức năng

| Feature | Status | Mô tả |
|---------|--------|--------|
| Stats Cards (Projects, Tasks, Issues) | 🟢 MVP | Tổng hợp số liệu |
| Recent Projects | 🟢 MVP | 3-4 project gần nhất |
| My Tasks (assigned to me) | 🟢 MVP | Danh sách task được giao |
| Activity Feed | 🟡 Scale | Timeline hoạt động gần đây |
| Charts (Progress) | 🟡 Scale | Biểu đồ tiến độ |
| Quick Actions | 🟢 MVP | Nút tạo nhanh Project/Task |

---

## 🧩 Components

| Component | Source |
|-----------|--------|
| `<StatsCard />` | features/dashboard |
| `<RecentProjects />` | features/dashboard |
| `<MyTasksList />` | features/dashboard |
| `<ActivityFeed />` | features/dashboard (Scale) |

## 🪝 Hooks

| Hook | Chức năng |
|------|----------|
| `useDashboardStats()` | Fetch tổng hợp stats |
| `useRecentProjects()` | Fetch recent projects |
| `useMyTasks()` | Fetch tasks assigned to current user |

## 📡 API Endpoints

| Method | Endpoint | Mô tả |
|--------|----------|--------|
| `GET` | `/dashboard/stats` | Tổng hợp số liệu |
| `GET` | `/projects?sort=recent&limit=4` | Recent projects |
| `GET` | `/tasks?assignee=me&limit=5` | My tasks |

---

*Last Updated: 2026-02-11*
