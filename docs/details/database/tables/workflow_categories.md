# workflow_categories Table

**Type:** Master Table (Metadata)  
**Tenant Isolation:** N/A (Single-Tenant)

---

## 📋 Schema

```sql
CREATE TABLE workflow_categories (
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
// models/workflow-category.model.ts
WorkflowCategory.belongsToMany(WorkflowTemplate, {
  through: 'workflow_category_mappings',
  foreignKey: 'category_id',
  otherKey: 'workflow_id',
  as: 'workflows'
});
```

---

*Last Updated: 2026-02-15*
