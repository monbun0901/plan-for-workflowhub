# project_categories Table

**Type:** Master Table (Metadata)  
**Tenant Isolation:** N/A (Single-Tenant)

---

## 📋 Schema

```sql
CREATE TABLE project_categories (
  id              VARCHAR(36) PRIMARY KEY,
  
  name            VARCHAR(100) NOT NULL,
  description     TEXT,
  order_index     INT DEFAULT 0,
  
  created_at      TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at      TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);
```

---

## 🔗 Associations (Sequelize)

```typescript
// models/project-category.model.ts
ProjectCategory.belongsToMany(Project, {
  through: 'project_category_mappings',
  foreignKey: 'category_id',
  otherKey: 'project_id',
  as: 'projects'
});
```

---

*Last Updated: 2026-02-15*
