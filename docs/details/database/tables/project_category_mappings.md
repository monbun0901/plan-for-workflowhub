# project_category_mappings Table

**Type:** Junction Table (Project ↔ Category)  
**Tenant Isolation:** N/A (Single-Tenant)

---

## 📋 Schema

```sql
CREATE TABLE project_category_mappings (
  project_id      VARCHAR(36) NOT NULL REFERENCES projects(id) ON DELETE CASCADE,
  category_id     VARCHAR(36) NOT NULL REFERENCES project_categories(id) ON DELETE CASCADE,
  
  PRIMARY KEY (project_id, category_id),
  INDEX idx_project (project_id),
  INDEX idx_category (category_id)
);
```

---

## 🔗 Associations (Sequelize)

```typescript
// models/project.model.ts
Project.belongsToMany(ProjectCategory, {
  through: 'project_category_mappings',
  foreignKey: 'project_id',
  otherKey: 'category_id',
  as: 'categories'
});
```

---

*Last Updated: 2026-02-15*
