# Pages UI/UX Specifications

**Version:** v1.1.0
**Date:** 2026-02-11

Tài liệu spec chi tiết cho từng page trong WorkflowHub, bao gồm layout, chức năng, và wireframe.

---

## 📊 Status Legend

| Icon | Phase | Mô tả |
|------|-------|--------|
| 🟢 | Phase 1 (MVP) | Implement ngay, đầy đủ spec |
| 🟡 | Scale-up | Có thiết kế, implement sau MVP |
| 🔴 | Coming (Phase 2) | Placeholder, chưa chi tiết |

---

## 🗺️ Page Map

### 🔐 Auth Pages
| Page | Route | Status | File |
|------|-------|--------|------|
| Login | `/login` | 🟢 MVP | [auth/01-login.md](auth/01-login.md) |
| Register | `/register` | 🟢 MVP | [auth/02-register.md](auth/02-register.md) |

### 📊 Dashboard
| Page | Route | Status | File |
|------|-------|--------|------|
| Overview | `/dashboard` | 🟢 MVP | [dashboard/01-overview.md](dashboard/01-overview.md) |

### 📁 Projects
| Page | Route | Status | File |
|------|-------|--------|------|
| Project List | `/projects` | 🟢 MVP | [dashboard/projects/01-project-list.md](dashboard/projects/01-project-list.md) |
| Create Project | `/projects/new` | 🟢 MVP | [dashboard/projects/02-project-create.md](dashboard/projects/02-project-create.md) |
| Project Detail | `/projects/:id` | 🟢 MVP | [dashboard/projects/03-project-detail.md](dashboard/projects/03-project-detail.md) |
| Edit Project | `/projects/:id/edit` | 🟢 MVP | [dashboard/projects/04-project-edit.md](dashboard/projects/04-project-edit.md) |

### ✅ Tasks
| Page | Route | Status | File |
|------|-------|--------|------|
| Task List | `/projects/:id/tasks` | 🟢 MVP | [dashboard/tasks/01-task-list.md](dashboard/tasks/01-task-list.md) |
| Task Detail | `/projects/:id/tasks/:taskId` | 🟢 MVP | [dashboard/tasks/02-task-detail.md](dashboard/tasks/02-task-detail.md) |
| Create Task | `/projects/:id/tasks/new` | 🟢 MVP | [dashboard/tasks/03-task-create.md](dashboard/tasks/03-task-create.md) |
| Edit Task | `/projects/:id/tasks/:taskId/edit` | 🟢 MVP | [dashboard/tasks/04-task-edit.md](dashboard/tasks/04-task-edit.md) |

### 🐛 Issues
| Page | Route | Status | File |
|------|-------|--------|------|
| Issue List | `/projects/:id/issues` | 🟢 MVP | [dashboard/issues/01-issue-list.md](dashboard/issues/01-issue-list.md) |
| Issue Detail | `/projects/:id/issues/:issueId` | 🟢 MVP | [dashboard/issues/02-issue-detail.md](dashboard/issues/02-issue-detail.md) |
| Create Issue | `/projects/:id/issues/new` | 🟢 MVP | [dashboard/issues/03-issue-create.md](dashboard/issues/03-issue-create.md) |

### 👥 Organization
| Page | Route | Status | File |
|------|-------|--------|------|
| Members | `/members` | 🟢 MVP | [dashboard/members/01-member-list.md](dashboard/members/01-member-list.md) |
| Invite Member | `/members/invite` | 🟢 MVP | [dashboard/members/02-member-invite.md](dashboard/members/02-member-invite.md) |
| Settings | `/settings` | 🟢 MVP | [dashboard/settings/01-settings.md](dashboard/settings/01-settings.md) |

### 🔄 Workflows
| Page | Route | Status | File |
|------|-------|--------|------|
| Workflow List | `/workflows` | 🟡 Scale | [dashboard/workflows/01-workflow-list.md](dashboard/workflows/01-workflow-list.md) |
| Workflow Editor | `/workflows/:id/edit` | 🟡 Scale | [dashboard/workflows/02-workflow-editor.md](dashboard/workflows/02-workflow-editor.md) |
| Workflow Template | `/workflows/templates/:id` | 🔴 Coming | [dashboard/workflows/03-workflow-template.md](dashboard/workflows/03-workflow-template.md) |

### 📑 Agents
| Page | Route | Status | File |
|------|-------|--------|------|
| Agent List | `/agents` | 🟡 Scale | [dashboard/agents/01-agent-list.md](dashboard/agents/01-agent-list.md) |
| Create Agent | `/agents/new` | 🟡 Scale | [dashboard/agents/02-agent-create.md](dashboard/agents/02-agent-create.md) |
| Edit Agent | `/agents/:id/edit` | 🟡 Scale | [dashboard/agents/03-agent-edit.md](dashboard/agents/03-agent-edit.md) |

### 📄 Documents
| Page | Route | Status | File |
|------|-------|--------|------|
| Document List | `/projects/:id/documents` | 🟡 Scale | [dashboard/documents/01-document-list.md](dashboard/documents/01-document-list.md) |
| Document Editor | `/documents/:id/edit` | 🟡 Scale | [dashboard/documents/02-document-editor.md](dashboard/documents/02-document-editor.md) |

### 💬 Chat with AI
| Page | Route | Status | File |
|------|-------|--------|------|
| Chat with AI | `/chat-with-ai` | 🔴 Coming | [dashboard/chat/01-chat-with-ai.md](dashboard/chat/01-chat-with-ai.md) |

---

## 📊 Summary

| Phase | Pages | Percentage |
|-------|-------|-----------|
| 🟢 MVP | 17 | 65% |
| 🟡 Scale-up | 7 | 27% |
| 🔴 Coming | 2 | 8% |
| **Total** | **26** | **100%** |

---

*Last Updated: 2026-02-11*
