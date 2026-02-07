# 📋 Kế Hoạch Dự Án WorkflowHub

> **Nền tảng quản lý công việc và dự án đa tổ chức tích hợp AI và workflow thông minh**

---

## 1. TỔNG QUAN DỰ ÁN

### Mô tả
WorkflowHub là nền tảng quản lý công việc đa tổ chức, tích hợp AI và workflow thông minh để tự động hóa quy trình làm việc.

### Đặc điểm nổi bật
- ✅ **Multi-tenant:** Hỗ trợ nhiều tổ chức độc lập
- ✅ **Smart AI:** AI với RAG + Agentic Workflow
- ✅ **Automation:** Workflow giảm thao tác thủ công
- ✅ **Scalability:** Kiến trúc module hóa, dễ mở rộng

---

## 2. SCOPE MVP

### 2.1 MVP Modules (Bắt buộc)

| Priority | Module | Mô tả |
|----------|--------|-------|
| 1 | **Projects** | Quản lý dự án, chứa Issues, Tasks, Documents |
| 2 | **Issues** | Hệ thống issue theo GitHub Style |
| 3 | **Tasks** | Nhiệm vụ cụ thể, thực thi từ Issue, có Deadline/Priority |
| 4 | **Documents** | Tài liệu & Knowledge Base cho AI |
| 5 | **Agents** | AI Agents tùy biến (PM, Dev, Reviewer) |
| 6 | **Workflow** | Quy trình tự động hóa |
| 7 | **Chat with AI** | Chat interface với AI (RAG-based) |

### 2.2 Edge Cases & Rules

#### Multi-tenant Isolation Rules
```
Organization A ──┬── Projects ──┬── Issues → Tasks
                 │             ├── Documents
                 │             ├── Workflows
                 │             └── AI Agents
                 │
Organization B ──┄── Projects ──┄── Issues → Tasks
                              ├── Documents
                              ├── Workflows
                              └── AI Agents

❌ Data không được leak giữa các Organization
✅ Agent chỉ truy cập data trong scope được phép
```

#### AI Agent Behavior Boundaries
- Agent chỉ hoạt động trong Organization/Project được assign
- Agent không thể truy cập data của Organization khác
- Agent chỉ có thể đọc/ghi data theo permissions được cấp
- System Prompt được define per Agent role

#### Workflow Trigger Conditions
- **Manual:** User trigger by button/action
- **Event-based:** Issue created/updated, Task completed
- **Scheduled:** Cron-like triggers
- **AI-triggered:** Agent action completes

---

## 3. PRIORITIES (Thứ tự ưu tiên)

### Development Order
```
Phase 1: Core AI Intelligence
         └── AI Agents (PM, Dev, Reviewer)

Phase 2: Automation Engine
         └── Workflow (Templates & Instances)

Phase 3: Knowledge Foundation
         └── Documents (Knowledge Base)

Phase 4: User Interface
         └── Chat with AI (RAG-based)
```

**Lưu ý:** Projects, Issues, Tasks là foundation - implement song song khi cần.

---

## 4. CÁC MODULES CHI TIẾT

### 4.1 Organizations (Tổ chức)
- Đại diện cho đơn vị / công ty / nhóm
- Quản lý: Members, Roles & Permissions, Projects

### 4.2 Members (Thành viên)
- Người dùng thuộc hệ thống
- Phân quyền theo RBAC / ABAC
- Cấp độ: `Owner` | `Admin` | `Member`

### 4.3 Projects (Dự án)
- Trung tâm điều phối của một dự án
- Chứa: Issues, Tasks, Documents

### 4.4 Issues (GitHub Style)
- Ghi nhận: Bug, Feature request, Improvement
- Thuộc tính: Labels, Status, Comments, Assignees

### 4.5 Tasks (Nhiệm vụ)
- Nhiệm vụ cụ thể, thực thi từ Issue
- Có Deadline, Priority
- Có thể được sinh tự động bởi AI Agent

### 4.6 Documents (Tài liệu)
- Hệ thống tri thức nội bộ (Markdown, Versioning)
- Nguồn dữ liệu đầu vào (Knowledge Base) chính cho AI

### 4.7 Agents (AI Agents)
- Tạo AI Agents tùy biến theo vai trò: PM, Dev, Reviewer
- Có System Prompt riêng và quyền truy cập dữ liệu được giới hạn

### 4.8 Workflow
- Định nghĩa quy trình làm việc tự động
- Gồm: Workflow Templates (thiết kế) và Workflow Instances (thực thi)

### 4.9 Chat with AI (Core Feature)
- AI phản hồi dựa trên RAG (Retrieval-Augmented Generation)
- Use cases:
  - "Dự án này còn task nào quá hạn?"
  - "Tóm tắt issue quan trọng"

---

## 5. MÔ HÌNH QUAN HỆ (HIERARCHY)

```
Organization
 ├── Members
 └── Projects
      ├── Issues ──> Tasks
      ├── Documents (Knowledge Base)
      ├── Workflows (Templates & Instances)
      └── AI Agents
```

---

## 6. TECH STACK

### Frontend
- **Framework:** Next.js (React)
- **UI Library:** TailwindCSS + shadcn/ui (recommended)

### Backend
- **Runtime:** Node.js
- **Framework:** NestJS hoặc Express.js
- **Database:** MySQL
- **ORM:** Prisma hoặc TypeORM

### AI & Vector
- **LLM:** OpenAI API / Anthropic API
- **Vector DB:** Pinecone / Weaviate / Chroma (cho RAG)
- **Embeddings:** OpenAI Embeddings

### Infrastructure
- **Deployment:** Docker + Docker Compose
- **CI/CD:** GitHub Actions
- **Hosting:** Vercel (Frontend) + Railway/Render (Backend)

---

## 7. LỘ TRÌNH PHÁT TRIỂN

### Phase 1: Core Foundation (2-3 tuần)
- [ ] Thiết kế database schema (MySQL)
- [ ] Authentication & Authorization (JWT + RBAC)
- [ ] Organizations & Members CRUD
- [ ] Basic API structure

### Phase 2: Project Management (2-3 tuần)
- [ ] Projects CRUD
- [ ] Issues system (GitHub style: Labels, Status, Assignees)
- [ ] Tasks management (Deadline, Priority)
- [ ] Comments & Notifications

### Phase 3: Documents & Knowledge Base (2 tuần)
- [ ] Document management (Markdown support)
- [ ] Versioning system
- [ ] RAG pipeline setup (Vector DB + Embeddings)

### Phase 4: AI Agents Core (3 tuần)
- [ ] Agent system architecture
- [ ] System Prompt templates (PM, Dev, Reviewer)
- [ ] Agent-Project assignment
- [ ] AI-generated tasks

### Phase 5: Workflow Automation (3 tuần)
- [ ] Workflow Templates designer
- [ ] Workflow Instances execution engine
- [ ] Triggers (Manual, Event, Scheduled)
- [ ] Conditions & Actions

### Phase 6: Chat with AI (2 tuần)
- [ ] Chat UI (Next.js)
- [ ] RAG integration
- [ ] Query processing
- [ ] Response generation

### Phase 7: Polish & Deploy (2 tuần)
- [ ] Dashboard & Analytics
- [ ] Responsive UI
- [ ] Unit & Integration tests
- [ ] Deployment & CI/CD

---

## 8. TIÊU CHÍ THÀNH CÔNG

- [ ] Multi-tenant isolation hoạt động đúng
- [ ] AI Agent thực hiện được task được giao
- [ ] Workflow tự động hóa thành công
- [ ] RAG trả lời chính xác câu hỏi từ knowledge base
- [ ] Hệ thống scale được khi tăng users

---

## 9. TÀI LIỆU THAM KHẢO

- Channel docs: 1469541799529025771
- Clarifying: Discord message 1469596817355182214
- Source: Discord #1469541799529025771

---

*Created: 2026-02-07*
*Last Updated: 2026-02-07*
