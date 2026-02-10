# agents Table

**Type:** AI Agent Registry  
**Tenant Isolation:** ✅ Required (`organization_id`)

---

## 📋 Schema

```sql
CREATE TABLE agents (
  id              VARCHAR(36) PRIMARY KEY,
  organization_id VARCHAR(36) NOT NULL REFERENCES organizations(id),
  project_id      VARCHAR(36) REFERENCES projects(id),  -- NULL = org-wide agent
  
  name            VARCHAR(100) NOT NULL,
  role            ENUM('pm', 'developer', 'reviewer', 'analyst', 'custom') NOT NULL,
  description     TEXT,
  
  -- AI Core Configuration
  system_prompt   TEXT NOT NULL,                -- "Cốt lõi" và hướng dẫn hành vi
  model           VARCHAR(50) DEFAULT 'gpt-4',  -- Mô hình mặc định
  tools           JSON,                         -- Danh sách công cụ được phép dùng
  document_ids    JSON,                         -- Nguồn tài liệu (Reference to documents.id)
  
  status          ENUM('active', 'disabled') DEFAULT 'active',
  
  created_at      TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at      TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  
  INDEX idx_org (organization_id),
  INDEX idx_project (project_id)
);
```

---

## 🎯 Purpose
Bảng này đóng vai trò là một **Danh mục (Registry)** các AI Agents khả dụng cho tổ chức/dự án. Backend không lưu cấu hình (Prompts, Models) mà chỉ lưu thông tin định danh để:
1.  **Hiển thị trên UI:** Cho phép người dùng chọn Agent để chat hoặc giao việc.
2.  **Tracking & Audit:** Biết bản ghi nào (Task/Comment) được tạo ra bởi Agent nào.
3.  **Routing:** Dùng `agent_external_key` để Backend biết phải gọi đến Service/Endpoint nào tương ứng.

---

## 🔗 Associations (Sequelize)

```typescript
// models/agent.model.ts
Agent.belongsTo(Organization, {
  foreignKey: 'organization_id',
  as: 'organization'
});

Agent.hasMany(Task, {
  foreignKey: 'generated_by_agent',
  as: 'generatedTasks'
});

Agent.hasMany(Chat, {
  foreignKey: 'agent_id',
  as: 'chats'
});
```

---

## 📝 Fields Explanation

| Field | Description | Example |
|-------|-------------|---------|
| `role` | Loại Agent | pm, developer, analyst |
| `agent_external_key` | Key định danh để gọi AI Service | 'pm-agent-standard', 'code-reviewer-v2' |
| `project_id` | Phạm vi hoạt động | Nếu có ID thì Agent chỉ dùng cho dự án đó |

---

## 📚 Related Tables

- **tasks** - Truy vết task được tạo bởi AI.
- **chats** - Các phiên hội thoại giữa User và Agent.
- **workflow_instances** - Agent có thể được gọi trong quá trình chạy automation.

---

*Last Updated: 2026-02-11*
