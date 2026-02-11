# workflow_statuses Table

**Type:** Lookup Table (Metadata)  
**Tenant Isolation:** ✅ Required (`organization_id`)

---

## 📋 Schema

```sql
CREATE TABLE workflow_statuses (
  id              VARCHAR(36) PRIMARY KEY,
  organization_id VARCHAR(36) NOT NULL REFERENCES organizations(id),
  
  name            VARCHAR(50) NOT NULL,         -- Ví dụ: "Đang làm", "Chờ duyệt"
  description     TEXT,
  order_index     INT DEFAULT 0,                -- Thứ tự hiển thị trên Kanban/List
  
  -- Logic mapping
  state_type      ENUM('todo', 'in_progress', 'done', 'closed') NOT NULL,
  
  -- Target entity
  target_type     ENUM('project', 'issue', 'task') NOT NULL,
  
  created_at      TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at      TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  
  INDEX idx_org_target_order (organization_id, target_type, order_index)
);
```

---

## 🎯 Purpose
Định nghĩa quy trình (Workflow) tùy chỉnh. Một tổ chức có thể có quy trình đơn giản (To Do -> Done) hoặc phức tạp (Open -> Researching -> Development -> QA -> Deploy -> Done).

**Ý nghĩa của `state_type`:**
Mặc dù `name` có thể là bất cứ thứ gì, hệ thống cần `state_type` để biết:
- `todo`: Task chưa bắt đầu.
- `in_progress`: Task đang được thực hiện.
- `done`: Task đã hoàn thành logic.
- `closed`: Task đã đóng hoàn toàn (Lưu trữ).

---

## 🔗 Associations (Sequelize)

```typescript
// models/workflow-status.model.ts
WorkflowStatus.belongsTo(Organization, {
  foreignKey: 'organization_id',
  as: 'organization'
});
```

---

## 🌱 Seed Data

### Default Statuses Per Organization

When a new organization is created, seed these default statuses:

```sql
-- Projects (3 states)
INSERT INTO workflow_statuses (organization_id, name, state_type, target_type, order_index)
VALUES 
  (orgId, 'Planning', 'todo', 'project', 0),
  (orgId, 'In Progress', 'in_progress', 'project', 1),
  (orgId, 'Completed', 'done', 'project', 2);

-- Issues (4 states)
INSERT INTO workflow_statuses (organization_id, name, state_type, target_type, order_index)
VALUES 
  (orgId, 'Open', 'todo', 'issue', 0),
  (orgId, 'In Progress', 'in_progress', 'issue', 1),
  (orgId, 'Fixed', 'done', 'issue', 2),
  (orgId, 'Closed', 'closed', 'issue', 3);

-- Tasks (3 states)
INSERT INTO workflow_statuses (organization_id, name, state_type, target_type, order_index)
VALUES 
  (orgId, 'Todo', 'todo', 'task', 0),
  (orgId, 'In Progress', 'in_progress', 'task', 1),
  (orgId, 'Done', 'done', 'task', 2);

-- Documents (3 states)
INSERT INTO workflow_statuses (organization_id, name, state_type, target_type, order_index)
VALUES 
  (orgId, 'Draft', 'todo', 'document', 0),
  (orgId, 'Published', 'done', 'document', 1),
  (orgId, 'Archived', 'closed', 'document', 2);

-- Workflow Templates (3 states)
INSERT INTO workflow_statuses (organization_id, name, state_type, target_type, order_index)
VALUES 
  (orgId, 'Draft', 'todo', 'workflow_template', 0),
  (orgId, 'Active', 'done', 'workflow_template', 1),
  (orgId, 'Archived', 'closed', 'workflow_template', 2);
```

---

## 🎯 Common Queries

### Get Kanban Columns for Projects

```typescript
const statuses = await WorkflowStatus.findAll({
  where: {
    organization_id: orgId,
    target_type: 'project'
  },
  order: [['order_index', 'ASC']]
});
```

### Get Document Statuses

```typescript
const statuses = await WorkflowStatus.findAll({
  where: {
    organization_id: orgId,
    target_type: 'document'
  },
  order: [['order_index', 'ASC']]
});
```

---

*Last Updated: 2026-02-11*
