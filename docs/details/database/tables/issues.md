# issues Table

**Type:** Core Business Entity  
**Tenant Isolation:** ✅ Required (`organization_id`)

---

## 📋 Schema

```sql
CREATE TABLE issues (
  id              VARCHAR(36) PRIMARY KEY,
  project_id      VARCHAR(36) NOT NULL REFERENCES projects(id),
  organization_id VARCHAR(36) NOT NULL REFERENCES organizations(id),
  
  number          INT NOT NULL,                  -- Auto-increment per project
  title           VARCHAR(200) NOT NULL,
  description     TEXT,
  
  -- Dynamic lookups (Nullable for initial creation)
  status_id       VARCHAR(36) REFERENCES workflow_statuses(id),
  -- Category dùng bảng trung gian issue_categories
  
  priority        ENUM('low', 'medium', 'high', 'critical') DEFAULT 'medium',
  visibility_id   VARCHAR(36) REFERENCES visibility_levels(id),
  
  -- Assignees dùng bảng trung gian issue_assignees hỗ trợ danh sách
  reporter_id     VARCHAR(36) NOT NULL REFERENCES users(id),
  
  -- Tags dùng bảng trung gian issue_tags (Thay thế labels)
  due_date        DATE,
  
  created_at      TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at      TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  closed_at       TIMESTAMP,
  
  UNIQUE KEY unique_project_number (project_id, number),
  INDEX idx_org_status (organization_id, status_id),
  INDEX idx_category (category_id)
);
```

---

## 🔗 Associations (Sequelize)

```typescript
// models/issue.model.ts
Issue.belongsTo(Project, {
  foreignKey: 'project_id',
  as: 'project'
});

Issue.belongsTo(WorkflowStatus, {
  foreignKey: 'status_id',
  as: 'status'
});

Issue.belongsToMany(Category, {
  through: 'issue_categories',
  foreignKey: 'issue_id',
  otherKey: 'category_id',
  as: 'categories'
});

Issue.belongsToMany(User, {
  through: 'issue_assignees',
  foreignKey: 'issue_id',
  otherKey: 'user_id',
  as: 'assignees'
});

Issue.belongsTo(User, {
  foreignKey: 'reporter_id',
  as: 'reporter'
});

Issue.hasMany(Task, {
  foreignKey: 'issue_id',
  as: 'tasks'
});

Issue.belongsToMany(Tag, {
  through: 'issue_tags',
  foreignKey: 'issue_id',
  otherKey: 'tag_id',
  as: 'tags'
});
```

**Explanation:**
- `belongsTo(Project)` - Issue thuộc 1 project.
- `belongsTo(WorkflowStatus)` - Trạng thái workflow.
- `belongsToMany(Category)` - Danh sách danh mục.
- `belongsToMany(User, assignees)` - Danh sách những người xử lý issue này.
- `belongsTo(User, reporter)` - Người tạo issue.
- `hasMany(Task)` - 1 issue có nhiều tasks con.

---

## 📝 Fields Explanation

| Field | Description | Example |
|-------|-------------|---------|
| `number` | Sequential number per project | #1, #2, #3 |
| `status_id` | Link to `workflow_statuses` | "Open", "Fixed" |
| `category_id` | Link to `categories` | "Bug", "Feature" |
| `priority` | Urgency level | low, medium, high, critical |
| `labels` | Tags for filtering | `['backend', 'security']` |

---

## 🎯 Common Queries

### List issues for project (with status & category)

```typescript
const issues = await Issue.findAll({
  where: {
    project_id: projectId,
    organization_id: orgId
  },
  include: [
    { model: WorkflowStatus, as: 'status' },
    { model: Category, as: 'category' },
    { model: User, as: 'assignee' },
    { model: User, as: 'reporter' }
  ],
  order: [['created_at', 'DESC']]
});
```

### Assign issue to user

```typescript
await Issue.update(
  { assignee_id: userId },
  { 
    where: { 
      id: issueId, 
      organization_id: orgId  // CRITICAL
    } 
  }
);
```

---

## 📚 Related Tables

- **projects** - Parent project
- **tasks** - Child tasks (break down issue)
- **users** - Assignee and reporter

---

*Last Updated: 2026-02-11*
