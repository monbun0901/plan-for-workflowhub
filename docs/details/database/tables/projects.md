# projects Table

**Type:** Core Business Entity  
**Tenant Isolation:** ✅ Required (`organization_id`)

---

## 📋 Schema

```sql
CREATE TABLE projects (
  id              VARCHAR(36) PRIMARY KEY,
  organization_id VARCHAR(36) NOT NULL REFERENCES organizations(id),
  
  name            VARCHAR(100) NOT NULL,
  slug            VARCHAR(100) NOT NULL,
  description     TEXT,
  
  -- Lookup dynamic data (Nullable for initial creation)
  status_id       VARCHAR(36) REFERENCES workflow_statuses(id),
  category_id     VARCHAR(36) REFERENCES categories(id),
  
  -- Dynamic Visibility
  visibility_id   VARCHAR(36) REFERENCES visibility_levels(id),
  
  -- Classification
  -- Tags nay dùng bảng trung gian project_tags liên kết với bảng tags (Master Data)
  
  settings        JSON,
  created_by      VARCHAR(36) REFERENCES users(id),
  created_at      TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at      TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  
  UNIQUE KEY unique_org_slug (organization_id, slug),
  INDEX idx_org_status (organization_id, status_id),
  INDEX idx_category (category_id)
);
```

---

## 🔑 Indexes

```sql
PRIMARY KEY (id)
UNIQUE KEY unique_org_slug (organization_id, slug)
CREATE INDEX idx_projects_org ON projects(organization_id);
CREATE INDEX idx_projects_org_status ON projects(organization_id, status_id);
CREATE INDEX idx_projects_category ON projects(category_id);
```

---

## 🔗 Associations (Sequelize)

```typescript
// models/project.model.ts
Project.belongsTo(Organization, {
  foreignKey: 'organization_id',
  as: 'organization'
});

Project.belongsTo(WorkflowStatus, {
  foreignKey: 'status_id',
  as: 'status'
});

Project.belongsTo(Category, {
  foreignKey: 'category_id',
  as: 'category'
});

Project.hasMany(ProjectMember, {
  foreignKey: 'project_id',
  as: 'members'
});

Project.hasMany(Issue, {
  foreignKey: 'project_id',
  as: 'issues'
});

Project.hasMany(Task, {
  foreignKey: 'project_id',
  as: 'tasks'
});

Project.belongsToMany(Tag, {
  through: 'project_tags',
  foreignKey: 'project_id',
  otherKey: 'tag_id',
  as: 'projectTags'
});

Project.belongsTo(User, {
  foreignKey: 'created_by',
  as: 'creator'
});
```

**Explanation:**
- `belongsTo(Organization)` - Project thuộc về 1 organization (tenant isolation).
- `belongsTo(WorkflowStatus)` - Trạng thái hiện tại của dự án (Dynamic).
- `belongsTo(Category)` - Phân loại dự án (Ví dụ: "Internal", "Client").
- `hasMany(ProjectMember)` - Danh sách Assignees/Members của dự án.
- `hasMany(Issue)` - 1 dự án có nhiều issues.
- `hasMany(Task)` - 1 dự án có nhiều tasks.
- `belongsTo(User)` - Người tạo dự án.

---

## 📝 Fields Explanation

| Field | Type | Description | Tenant Isolation |
|-------|------|-------------|------------------|
| `organization_id` | UUID | Owner organization | ✅ CRITICAL |
| `name` | VARCHAR(100) | Project name | - |
| `slug` | VARCHAR(100) | URL-friendly ID (unique per org) | - |
| `status_id` | UUID | Link to `workflow_statuses` table | - |
| `category_id` | UUID | Link to `categories` table | - |
| `visibility_id` | UUID | Link to `visibility_levels` table | - |
| `settings` | JSON | Project config (workflows, AI settings) | - |

### settings JSON Structure (Ví dụ)
Dùng để lưu các tùy chỉnh đặc thù cho từng dự án:
- `features`: `{ "kanban": true, "gantt": false }`
- `notifications`: `{ "slack_webhook": "...", "overdue_reminder": true }`
- `ai_rules`: Các chỉ thị AI dành riêng cho project này.

---

## 🎯 Common Queries

### List projects for organization (with status & category)

```typescript
const projects = await Project.findAll({
  where: {
    organization_id: orgId
  },
  include: [
    { model: WorkflowStatus, as: 'status' },
    { model: Category, as: 'category' }
  ],
  order: [['created_at', 'DESC']]
});
```

### Get project with detailed members (Assignees)

```typescript
const project = await Project.findOne({
  where: { id: projectId, organization_id: orgId },
  include: [
    { 
      model: ProjectMember, 
      as: 'members',
      include: [{ model: User, as: 'user' }] 
    }
  ]
});
```

### Filter projects by tags

```typescript
// Find all frontend projects
const frontendProjects = await Project.findAll({
  where: {
    organization_id: orgId,
    tags: {
      [Op.contains]: ['frontend']
    }
  },
  include: [{ model: WorkflowStatus, as: 'status' }]
});
```


---

## 🔒 Tenant Isolation Rules

```typescript
// ❌ DANGEROUS: No tenant filter
const project = await Project.findByPk(projectId);

// ✅ CORRECT: Always include organization_id
const project = await Project.findOne({
  where: { id: projectId, organization_id: orgId }
});
```

---

## ✅ Validation Rules

- `name`: 1-100 characters
- `slug`: Generated from name, unique within organization
- `organization_id`: REQUIRED, validated via middleware

---

## 📚 Related Tables

- **organizations** - Parent organization
- **issues** - Child issues
- **tasks** - Child tasks
- **documents** - Project documentation
- **agents** - Project-specific AI agents (optional)

---

*Last Updated: 2026-02-11*
