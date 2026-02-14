# tasks Table

**Type:** Core Business Entity  
**Tenant Isolation:** N/A (Single-Tenant)

---

## 📋 Schema

```sql
CREATE TABLE tasks (
  id              VARCHAR(36) PRIMARY KEY,
  issue_id        VARCHAR(36) REFERENCES issues(id),
  project_id      VARCHAR(36) NOT NULL REFERENCES projects(id),
  
  number          INT NOT NULL,                  -- Auto-increment per project
  title           VARCHAR(200) NOT NULL,
  description     TEXT,
  
  status_id       VARCHAR(36) REFERENCES task_statuses(id), -- Default: "Backlog/To Do" (Init by App)
  
  visibility      ENUM('public', 'private', 'restricted') DEFAULT 'public',

  created_by      VARCHAR(36) REFERENCES users(id),
  
  -- AI-generated tracking
  ai_generated    BOOLEAN DEFAULT FALSE,
  generated_by_agent VARCHAR(36) REFERENCES agents(id),
  
  created_at      TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at      TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  
  UNIQUE KEY unique_project_number (project_id, number),
  INDEX idx_status (status_id),
  INDEX idx_project (project_id),
  INDEX idx_issue (issue_id)
);
```

---

## 📝 Fields Explanation

| Field | Description | Note |
|-------|-------------|------|
| `number` | Số thứ tự định danh | Tự động tăng (Incremental) cho mỗi Project. Ví dụ: #1, #2... Giúp người dùng dễ gọi tên Task thay vì dùng UUID. |

---

## 🔗 Associations (Sequelize)

```typescript
// models/task.model.ts
Task.belongsTo(Issue, { foreignKey: 'issue_id', as: 'issue' });
Task.belongsTo(Project, { foreignKey: 'project_id', as: 'project' });
Task.belongsTo(TaskStatus, { foreignKey: 'status_id', as: 'status' });

Task.belongsToMany(TaskCategory, {
  through: 'task_category_mappings',
  foreignKey: 'task_id',
  otherKey: 'category_id',
  as: 'categories'
});

Task.belongsToMany(User, {
  through: 'task_assignees',
  foreignKey: 'task_id',
  otherKey: 'user_id',
  as: 'assignees'
});

Task.belongsTo(User, { foreignKey: 'created_by', as: 'creator' });
Task.belongsTo(Agent, { foreignKey: 'generated_by_agent', as: 'agent' });
```

---

## 🎯 Common Queries

### Kanban board view (Filtered by Project)

```typescript
const tasks = await Task.findAll({
  where: { project_id: projectId },
  include: [
    { model: TaskStatus, as: 'status' },
    { model: TaskCategory, as: 'categories' },
    { model: User, as: 'assignees' }
  ],
  order: [['created_at', 'ASC']]
});
```

---

*Last Updated: 2026-02-15*
