# tasks Table

**Type:** Core Business Entity  
**Tenant Isolation:** ✅ Required (`organization_id`)

---

## 📋 Schema

```sql
CREATE TABLE tasks (
  id              VARCHAR(36) PRIMARY KEY,
  issue_id        VARCHAR(36) REFERENCES issues(id),
  project_id      VARCHAR(36) NOT NULL REFERENCES projects(id),
  organization_id VARCHAR(36) NOT NULL REFERENCES organizations(id),
  
  title           VARCHAR(200) NOT NULL,
  description     TEXT,
  
  -- Dynamic lookups (Nullable for initial creation)
  status_id       VARCHAR(36) REFERENCES workflow_statuses(id),
  -- Category nay dùng bảng trung gian task_categories để hỗ trợ danh sách (Multi-select)
  
  priority        ENUM('low', 'medium', 'high', 'critical') DEFAULT 'medium',
  visibility_id   VARCHAR(36) REFERENCES visibility_levels(id),
  
  -- Tags dùng bảng trung gian task_tags (Thay thế labels)
  
  -- Assignees dùng bảng trung gian task_assignees hỗ trợ danh sách (Multi-assignee)
  created_by      VARCHAR(36) REFERENCES users(id),  -- NULL if AI-generated (Scale-up)
  
  estimated_hours DECIMAL(5,2),
  actual_hours    DECIMAL(5,2),
  due_date        DATETIME,
  completed_at    TIMESTAMP,
  
  -- AI-generated tracking (Phase 2: Scale-up)
  ai_generated    BOOLEAN DEFAULT FALSE,
  generated_by_agent VARCHAR(36) REFERENCES agents(id),
  
  created_at      TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at      TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  
  INDEX idx_org_status (organization_id, status_id),
  INDEX idx_category (category_id),
  INDEX idx_assignee (assignee_id)
);
```

**Logic:**
- `created_by = NULL` + `ai_generated = TRUE` → AI tạo
- `created_by = user_id` + `ai_generated = FALSE` → User tạo manual


---

## 🔗 Associations (Sequelize)

```typescript
// models/task.model.ts
Task.belongsTo(Issue, {
  foreignKey: 'issue_id',
  as: 'issue'
});

Task.belongsTo(Project, {
  foreignKey: 'project_id',
  as: 'project'
});

Task.belongsTo(WorkflowStatus, {
  foreignKey: 'status_id',
  as: 'status'
});

Task.belongsToMany(Category, {
  through: 'task_categories',
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

Task.belongsTo(User, {
  foreignKey: 'created_by',
  as: 'creator'
});

Task.belongsTo(Agent, {
  foreignKey: 'generated_by_agent',
  as: 'agent'
});
```

**Explanation:**
- `belongsTo(Issue)` - Task có thể thuộc 1 issue (optional).
- `belongsTo(Project)` - Task thuộc 1 project (required).
- `belongsTo(WorkflowStatus)` - Trạng thái workflow.
- `belongsToMany(Category)` - Danh sách danh mục.
- `belongsToMany(User, assignees)` - Danh sách những người đang thực hiện task.
- `belongsTo(User, creator)` - Người tạo task.
Task.belongsToMany(Tag, {
  through: 'task_tags',
  foreignKey: 'task_id',
  otherKey: 'tag_id',
  as: 'tags'
});

---

## 📝 Fields Explanation

| Field | Description | AI Context |
|-------|-------------|------------|
| `issue_id` | Parent issue (nullable) | Task có thể standalone |
| `status_id` | Link to `workflow_statuses` | "To Do", "Done" |
| `category_id` | Link to `categories` | "Backend", "Docs" |
| `labels` | Tags for classification | `['frontend', 'refactor', 'urgent']` |
| `ai_generated` | Created by AI? | TRUE nếu AI tạo |
| `generated_by_agent` | Which AI agent | PM, Developer, etc. |
| `estimated_hours` | Time estimate | AI có thể suggest |
| `actual_hours` | Time spent | For tracking |

**Labels Usage:**
- **Tech**: `frontend`, `backend`, `testing`, `devops`
- **Type**: `refactor`, `bugfix`, `feature`, `documentation`
- **Priority**: `urgent`, `blocking`, `nice-to-have`
- **Status**: `reviewed`, `wip`, `needs-testing`

---

## 🎯 Common Queries

### Kanban board view (Filtered by Project)

```typescript
const tasks = await Task.findAll({
  where: {
    project_id: projectId,
    organization_id: orgId
  },
  include: [
    { model: WorkflowStatus, as: 'status' },
    { model: Category, as: 'category' },
    { model: User, as: 'assignee' }
  ],
  order: [['created_at', 'ASC']]
});
```

### AI-generated tasks

```typescript
const aiTasks = await Task.findAll({
  where: {
    organization_id: orgId,
    ai_generated: true
  },
  include: [{ model: Agent, as: 'agent' }]
});
```

---

## 📚 Related Tables

- **issues** - Parent issue (optional)
- **projects** - Parent project
- **agents** - AI that generated task
- **users** - Assignee and creator
- **task_assignments** - Assignment history (recommended)

**Note:** Recommend creating separate `task_assignments` table to track:
- Assignment history (who → who)
- Who assigned the task
- Duration per assignee
- Reason for reassignment

See [task_assignments.md](task_assignments.md) for full details.

---

*Last Updated: 2026-02-11*
