# Step 2: MVP Scope & Requirements Clarification

**Date:** 2026-02-07  
**Status:** ✅ Completed

## Sources

- Original Docs: [v1-initial-plan.md](v1-initial-plan.md)
- Discord: Message 1469596817355182214

---

## 1. MVP Scope (Bắt buộc)

| Priority | Module | Description |
|----------|--------|-------------|
| 1 | **Projects** | Quản lý dự án, chứa Issues, Tasks, Documents |
| 2 | **Issues** | Hệ thống issue theo GitHub Style |
| 3 | **Tasks** | Nhiệm vụ cụ thể, thực thi từ Issue, có Deadline/Priority |
| 4 | **Documents** | Tài liệu & Knowledge Base cho AI |
| 5 | **Agents** | AI Agents tùy biến (PM, Dev, Reviewer) |
| 6 | **Workflow** | Quy trình tự động hóa |
| 7 | **Chat with AI** | Chat interface với AI (RAG-based) |

---

## 2. Development Priorities

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

**Note:** Projects, Issues, Tasks - implement song song khi cần.

---

## 3. Tech Stack

| Layer | Technology |
|-------|------------|
| Frontend | Next.js |
| Backend | Node.js |
| Database | MySQL |
| AI | OpenAI/Anthropic API |
| Vector DB | Pinecone/Weaviate/Chroma |

---

## 4. Edge Cases & Business Rules

### 4.1 Multi-tenant Isolation

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
```

**Rules:**
- ❌ Data không được leak giữa các Organization
- ✅ Agent chỉ truy cập data trong scope được phép

### 4.2 AI Agent Behavior Boundaries

- Agent chỉ hoạt động trong Organization/Project được assign
- Agent không thể truy cập data của Organization khác
- Agent chỉ có thể đọc/ghi data theo permissions được cấp
- System Prompt được define per Agent role

### 4.3 Workflow Trigger Conditions

| Trigger Type | Description |
|-------------|-------------|
| **Manual** | User trigger by button/action |
| **Event-based** | Issue created/updated, Task completed |
| **Scheduled** | Cron-like triggers |
| **AI-triggered** | Agent action completes |

---

## 5. Related Issues

- #1: Tạo repo và lập kế hoạch WorkflowHub
- #2: [Step 2] Phân tích & Làm rõ yêu cầu MVP

---

*Document Version: 1.0*
*Last Updated: 2026-02-07*
