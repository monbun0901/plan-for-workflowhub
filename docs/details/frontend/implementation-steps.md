# Các Bước Xây Dựng Frontend

**Framework:** Next.js 14+ (App Router)  
**Styling:** TailwindCSS  
**State:** Zustand  
**API:** Axios  
**Validation:** Zod

> Mỗi bước tương ứng với 1 phase backend.  
> Chỉ bắt đầu bước N khi backend phase tương ứng đã sẵn sàng.

---

## 📋 Tổng quan 12 bước

| Bước | Nội dung | Tương ứng Backend |
|------|----------|-------------------|
| 1 | Khởi tạo dự án | Phase 1 (Infra) |
| 2 | Hệ thống Layout | Phase 1 |
| 3 | Xác thực (Auth) | Phase 2 |
| 4 | Tổ chức & Thành viên | Phase 3 |
| 5 | Dự án (Projects) | Phase 4 |
| 6 | Vấn đề (Issues) | Phase 4 |
| 7 | Nhiệm vụ (Tasks) | Phase 4 |
| 8 | Tài liệu (Documents) | Phase 5 |
| 9 | Dữ liệu nền (Master Data) | Phase 4 |
| 10 | AI Chat & Agents | Phase 6 |
| 11 | Dashboard & Analytics | Phase 8 |
| 12 | Hoàn thiện | Phase 8 |

---

## Bước 1: Khởi tạo dự án

**Mục tiêu:** Setup Next.js + TailwindCSS + Zustand + Axios + cấu trúc thư mục

### Việc cần làm

```bash
# Khởi tạo Next.js trong apps/web
[ ] npx create-next-app@latest apps/web --typescript --tailwind --eslint --app
[ ] Cấu hình pnpm workspace (apps/web liên kết packages/shared)
[ ] Cài đặt dependencies: zustand, axios, zod, lucide-react
[ ] Tạo cấu trúc thư mục chuẩn (xem bên dưới)
[ ] Cấu hình Axios instance (baseURL, interceptors)
[ ] Tạo file constants (API endpoints, routes, colors)
```

### Cấu trúc thư mục

```
apps/web/src/
├── app/                       # Next.js App Router
│   ├── (auth)/               # Route group: Login, Register
│   ├── (dashboard)/          # Route group: Tất cả trang chính
│   ├── layout.tsx            # Root layout
│   └── globals.css
├── components/
│   ├── ui/                   # Atomic components (Button, Input, Modal)
│   ├── layout/               # Header, Sidebar, AppLayout
│   └── shared/               # Shared components (Avatar, Badge, etc.)
├── hooks/                    # Custom hooks
├── services/                 # Axios API services
├── stores/                   # Zustand stores
├── constants/                # API URLs, routes, enums
├── types/                    # TypeScript types & Zod schemas
└── utils/                    # Helper functions
```

### Tham khảo
- [01-architecture.md](./01-architecture.md) — Frontend architecture
- [constants/](./constants/) — Constants specs

---

## Bước 2: Hệ thống Layout

**Mục tiêu:** Tạo khung giao diện chính (Sidebar + Header + Content)

### Việc cần làm

```bash
[ ] Tạo AppLayout (sidebar + header + main content)
[ ] Tạo Sidebar (navigation menu, org selector, collapsible)
[ ] Tạo Header (breadcrumb, user menu, notifications)
[ ] Tạo AuthLayout (layout riêng cho login/register)
[ ] Responsive: mobile sidebar → drawer
```

### Components tạo mới

| Component | File | Mô tả |
|-----------|------|-------|
| `AppLayout` | `components/layout/AppLayout.tsx` | Layout chính (sidebar + content) |
| `Sidebar` | `components/layout/Sidebar.tsx` | Menu điều hướng |
| `Header` | `components/layout/Header.tsx` | Header với breadcrumb |
| `AuthLayout` | `components/layout/AuthLayout.tsx` | Layout trang auth |

### Tham khảo
- [layouts/](./layouts/) — Layout specs chi tiết

---

## Bước 3: Xác thực (Authentication)

**Mục tiêu:** Login, Register, Auth state, Protected routes

### Việc cần làm

```bash
# Trang
[ ] /login — Form đăng nhập (email + password)
[ ] /register — Form đăng ký (name + email + password)

# Store
[ ] authStore (Zustand) — user, tokens, isAuthenticated, login(), logout()

# Service
[ ] authService (Axios) — POST /auth/register, POST /auth/login, GET /auth/me

# Middleware
[ ] Auth guard — redirect to /login nếu chưa đăng nhập
[ ] Axios interceptor — tự động gắn JWT, refresh token khi hết hạn
```

### Tham khảo
- [pages/auth/](./pages/auth/) — Auth page specs
- [stores/01-auth-store.md](./stores/01-auth-store.md)

---

## Bước 4: Tổ chức & Thành viên

**Mục tiêu:** Chọn org, quản lý thành viên, mời người mới

### Việc cần làm

```bash
# Trang
[ ] Organization Selector (dropdown trên sidebar)
[ ] /settings/organization — Cài đặt tổ chức
[ ] /settings/members — Danh sách thành viên + mời

# Store
[ ] organizationStore — currentOrg, organizations[], switchOrg()

# Service
[ ] organizationService — CRUD organizations
[ ] memberService — list, invite, remove, updateRole

# Components
[ ] OrgSelectorDropdown — Chuyển đổi tổ chức
[ ] InviteMemberModal — Form mời thành viên (email + role)
[ ] MemberList — Bảng danh sách + actions (remove, change role)
```

### Tham khảo
- [stores/02-organization-store.md](./stores/02-organization-store.md)

---

## Bước 5: Dự án (Projects)

**Mục tiêu:** CRUD dự án, settings, thành viên dự án

### Việc cần làm

```bash
# Trang
[ ] /projects — Danh sách dự án (grid/list view)
[ ] /projects/new — Tạo dự án mới
[ ] /projects/[slug] — Chi tiết dự án (tabs: Issues, Tasks, Docs, Settings)
[ ] /projects/[slug]/settings — Cài đặt dự án

# Store
[ ] projectStore — projects[], currentProject, filters

# Service
[ ] projectService — CRUD + list members + add/remove members

# Components
[ ] ProjectCard — Card hiển thị dự án
[ ] ProjectForm — Form tạo/sửa dự án
[ ] ProjectTabs — Tab navigation (Issues | Tasks | Documents | Settings)
```

### Tham khảo
- [pages/dashboard/](./pages/dashboard/) — Dashboard page specs

---

## Bước 6: Vấn đề (Issues)

**Mục tiêu:** GitHub-style issue tracking

### Việc cần làm

```bash
# Trang
[ ] /projects/[slug]/issues — Danh sách issues (filter, search, sort)
[ ] /projects/[slug]/issues/new — Tạo issue mới
[ ] /projects/[slug]/issues/[number] — Chi tiết issue + comments

# Store
[ ] issueStore — issues[], filters, pagination

# Service
[ ] issueService — CRUD + list + filter + assignees

# Components
[ ] IssueList — Danh sách issues (table hoặc list)
[ ] IssueDetail — Chi tiết với sidebar (status, assignee, labels)
[ ] IssueForm — Form tạo/sửa issue
[ ] CommentThread — Danh sách comments + form thêm comment
[ ] StatusBadge — Hiển thị trạng thái (color-coded)
[ ] AssigneeSelector — Chọn người được gán
```

---

## Bước 7: Nhiệm vụ (Tasks)

**Mục tiêu:** Kanban board + task management

### Việc cần làm

```bash
# Trang
[ ] /projects/[slug]/tasks — Kanban board (drag & drop theo status)
[ ] Task detail — Modal hoặc side panel
[ ] Task create/edit — Form trong modal

# Store
[ ] taskStore — tasks[], kanbanColumns, moveTask()

# Service
[ ] taskService — CRUD + update status + assign + time tracking

# Components
[ ] KanbanBoard — Các cột theo workflow_status
[ ] KanbanColumn — 1 cột (header + danh sách cards)
[ ] TaskCard — Card hiển thị task (title, assignee, priority, due date)
[ ] TaskDetailPanel — Slide-over panel chi tiết task
[ ] TaskForm — Form tạo/sửa task
[ ] TimeTracker — Hiển thị estimated_hours / actual_hours
```

### Tham khảo
- [components/](./components/) — Component specs

---

## Bước 8: Tài liệu (Documents)

**Mục tiêu:** Knowledge base với Markdown editor

### Việc cần làm

```bash
# Trang
[ ] /projects/[slug]/docs — Document tree (sidebar)
[ ] /projects/[slug]/docs/[docSlug] — Document viewer
[ ] /projects/[slug]/docs/[docSlug]/edit — Markdown editor

# Store
[ ] documentStore — documents[], tree structure, currentDoc

# Service
[ ] documentService — CRUD + versioning + search

# Components
[ ] DocumentTree — Tree navigation (parent_id hierarchy)
[ ] MarkdownEditor — Editor (dùng thư viện: react-markdown + editor)
[ ] MarkdownViewer — Render markdown content
[ ] VersionHistory — Danh sách phiên bản + compare
[ ] EmbeddingStatus — Badge hiển thị trạng thái embedding
```

---

## Bước 9: Dữ liệu nền (Master Data)

**Mục tiêu:** Quản lý tags, categories, statuses

### Việc cần làm

```bash
# Trang
[ ] /settings/tags — CRUD tags (tên + màu)
[ ] /settings/categories — CRUD categories
[ ] /settings/statuses — Quản lý workflow statuses per target_type

# Service
[ ] masterDataService — CRUD tags, categories, statuses

# Components
[ ] TagManager — Danh sách + form tạo tag (có color picker)
[ ] CategoryManager — Danh sách + form tạo category
[ ] StatusManager — Quản lý trạng thái theo target type
[ ] ColorPicker — Component chọn màu cho tags
```

---

## Bước 10: AI Chat & Agents

**Mục tiêu:** Chat với AI, quản lý agents

### Việc cần làm

```bash
# Trang
[ ] /projects/[slug]/chat — Danh sách cuộc trò chuyện
[ ] /projects/[slug]/chat/[chatId] — Cửa sổ chat
[ ] /settings/agents — Quản lý AI agents

# Store
[ ] chatStore — chats[], currentChat, messages[], sendMessage()

# Service
[ ] chatService — create chat, send message, list messages
[ ] agentService — CRUD agents

# Components
[ ] ChatSidebar — Danh sách cuộc trò chuyện
[ ] ChatWindow — Cửa sổ tin nhắn (scroll, loading)
[ ] MessageBubble — Tin nhắn user/AI (có source citations)
[ ] MessageInput — Input + send button + loading state
[ ] AgentSelector — Chọn AI agent cho cuộc trò chuyện
[ ] SourceCitation — Hiển thị nguồn tài liệu AI trích dẫn
[ ] AgentForm — Form tạo/sửa agent (name, role, system_prompt)
```

### Tham khảo
- [pages/dashboard/](./pages/dashboard/) — Chat page specs

---

## Bước 11: Dashboard & Analytics

**Mục tiêu:** Tổng quan dự án, biểu đồ, activity feed

### Việc cần làm

```bash
# Trang
[ ] / (Dashboard home) — Tổng quan tất cả dự án
[ ] /projects/[slug] — Project overview dashboard

# Components
[ ] StatsCard — Card thống kê (số issues, tasks, docs)
[ ] ActivityFeed — Danh sách hoạt động gần đây
[ ] ProjectProgressChart — Biểu đồ tiến độ (burndown/pie)
[ ] QuickActions — Nút tạo nhanh (New Issue, New Task, New Doc)
```

---

## Bước 12: Hoàn thiện (Polish)

**Mục tiêu:** Responsive, UX, performance

### Việc cần làm

```bash
# Responsive Design
[ ] Mobile sidebar → drawer/overlay
[ ] Table → card view trên mobile
[ ] Modal → fullscreen trên mobile

# UX States
[ ] Loading skeletons cho mọi trang
[ ] Empty states (khi chưa có dữ liệu)
[ ] Error states (khi API lỗi)
[ ] Toast notifications (thành công/lỗi)

# Performance
[ ] React.lazy() cho routes (code splitting)
[ ] Image optimization (next/image)
[ ] Debounce search inputs

# SEO & Accessibility
[ ] Meta tags cho mỗi trang
[ ] ARIA labels cho interactive elements
[ ] Keyboard navigation
```

### Tham khảo
- [06-best-practices.md](./06-best-practices.md) — Best practices
- [08-frontend-standards.md](./08-frontend-standards.md) — Coding standards

---

## 📚 Tài liệu liên quan

- [Frontend Architecture](./01-architecture.md) — Kiến trúc frontend
- [State Management](./02-state-management.md) — Zustand patterns
- [API Service](./03-api-service.md) — Axios setup
- [Backend API Routes](../backend/api-routes-map.md) — Danh sách endpoints
- [Backend DTOs](../backend/dtos/README.md) — Request/Response schemas
- [Step 7 - Implementation Plan](../../basics/step-7-implementation-plan.md) — Kế hoạch tổng thể

---

*Cập nhật lần cuối: 2026-02-13*
