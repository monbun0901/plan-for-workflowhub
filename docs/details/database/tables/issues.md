# issues Table

**Type:** Core Business Entity  
**Tenant Isolation:** N/A (Single-Tenant)

---

## 📋 Schema

```sql
CREATE TABLE issues (
  id              VARCHAR(36) PRIMARY KEY,
  project_id      VARCHAR(36) NOT NULL REFERENCES projects(id),
  
  number          INT NOT NULL,                  -- Auto-increment per project
  title           VARCHAR(200) NOT NULL,
  description     TEXT,
  
  -- Lookup dynamic data
  status_id       VARCHAR(36) REFERENCES issue_statuses(id), -- Default: "Open" (Init by App)
  
  visibility      ENUM('public', 'private', 'restricted') DEFAULT 'public',
  
  -- Assignees dùng bảng trung gian issue_assignees hỗ trợ danh sách
  reporter_id     VARCHAR(36) NOT NULL REFERENCES users(id),
  
  created_at      TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at      TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  
  UNIQUE KEY unique_project_number (project_id, number),
  INDEX idx_status (status_id),
  INDEX idx_project (project_id)
);
```

---

## � Fields Explanation

| Field | Description | Note |
|-------|-------------|------|
| `number` | Số thứ tự định danh | Tự động tăng (Incremental) cho mỗi Project. Ví dụ: #1, #2... Giúp người dùng dễ gọi tên Issue thay vì dùng UUID. |

---

## �🔗 Associations (Sequelize)

```typescript
// models/issue.model.ts
Issue.belongsTo(Project, { foreignKey: 'project_id', as: 'project' });
Issue.belongsTo(IssueStatus, { foreignKey: 'status_id', as: 'status' });

Issue.belongsToMany(User, {
  through: 'issue_assignees',
  foreignKey: 'issue_id',
  otherKey: 'user_id',
  as: 'assignees'
});

Issue.belongsTo(User, { foreignKey: 'reporter_id', as: 'reporter' });
Issue.hasMany(Task, { foreignKey: 'issue_id', as: 'tasks' });
```

---

## 🎯 Common Queries

### List issues for project

```typescript
const issues = await Issue.findAll({
  where: { project_id: projectId },
  include: [
    { model: IssueStatus, as: 'status' },
    { model: User, as: 'assignees' }
  ],
  order: [['created_at', 'DESC']]
});
```

---

*Last Updated: 2026-02-15*
