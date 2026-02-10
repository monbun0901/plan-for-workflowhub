# project_tags Table

**Type:** Junction Table (Project ↔ Tag)  
**Tenant Isolation:** ✅ Required (`organization_id`)

---

## 📋 Schema

```sql
CREATE TABLE project_tags (
  project_id      VARCHAR(36) NOT NULL REFERENCES projects(id) ON DELETE CASCADE,
  tag_id          VARCHAR(36) NOT NULL REFERENCES tags(id) ON DELETE CASCADE,
  organization_id VARCHAR(36) NOT NULL REFERENCES organizations(id),
  
  PRIMARY KEY (project_id, tag_id),
  INDEX idx_project (project_id),
  INDEX idx_tag (tag_id)
);
```

---

## 🎯 Purpose
Liên kết các dự án với danh sách nhãn dãn tập trung của tổ chức.

---

## 🔗 Associations (Sequelize)

```typescript
// models/project.model.ts
Project.belongsToMany(Tag, {
  through: 'project_tags',
  foreignKey: 'project_id',
  otherKey: 'tag_id',
  as: 'projectTags'
});
```

---

*Last Updated: 2026-02-11*
