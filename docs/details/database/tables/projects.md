# projects Table

**Type:** Core Business Entity  
**Tenant Isolation:** N/A (Single-Tenant)

---

## 📋 Schema

```sql
CREATE TABLE projects (
  id              VARCHAR(36) PRIMARY KEY,
  
  name            VARCHAR(100) NOT NULL,
  slug            VARCHAR(100) UNIQUE NOT NULL, -- URL-friendly ID
  description     TEXT,
  
  -- Lookup dynamic data
  status_id       VARCHAR(36) REFERENCES project_statuses(id),
  
  -- Classification & Access
  visibility      ENUM('public', 'private', 'restricted') DEFAULT 'public',
  
  settings        JSON,
  created_by      VARCHAR(36) REFERENCES users(id),
  
  created_at      TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at      TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);
```

---

## 🔑 Indexes

```sql
PRIMARY KEY (id)
UNIQUE KEY unique_slug (slug)
CREATE INDEX idx_projects_status ON projects(status_id);
```

---

## 🔗 Associations (Sequelize)

```typescript
// models/project.model.ts
Project.belongsTo(ProjectStatus, { foreignKey: 'status_id', as: 'status' });

Project.belongsToMany(ProjectCategory, {
  through: 'project_category_mappings',
  foreignKey: 'project_id',
  otherKey: 'category_id',
  as: 'categories'
});

Project.hasMany(Issue, { foreignKey: 'project_id', as: 'issues' });
Project.hasMany(Task, { foreignKey: 'project_id', as: 'tasks' });
Project.belongsTo(User, { foreignKey: 'created_by', as: 'creator' });
```

---

## 📝 Fields Explanation

| Field | Type | Description |
|-------|------|-------------|
| `name` | VARCHAR(100) | Project name |
| `slug` | VARCHAR(100) | URL-friendly ID (Must be unique) |
| `status_id` | UUID | Default status (Init by App) |
| `visibility` | ENUM | public, private, restricted (Default: public) |
| `settings` | JSON | Project config (workflows, AI settings) |

---

## 🎯 Common Queries

### List all projects with status

```typescript
const projects = await Project.findAll({
  include: [
    { model: ProjectStatus, as: 'status' },
    { model: ProjectCategory, as: 'categories' }
  ],
  order: [['created_at', 'DESC']]
});
```

---

*Last Updated: 2026-02-15*
