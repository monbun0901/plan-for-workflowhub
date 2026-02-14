# workflow_category_mappings Table

**Type:** Junction Table (Workflow ↔ Category)  
**Tenant Isolation:** N/A (Single-Tenant)

---

## 📋 Schema

```sql
CREATE TABLE workflow_category_mappings (
  workflow_id     VARCHAR(36) NOT NULL REFERENCES workflow_templates(id) ON DELETE CASCADE,
  category_id     VARCHAR(36) NOT NULL REFERENCES workflow_categories(id) ON DELETE CASCADE,
  
  PRIMARY KEY (workflow_id, category_id),
  INDEX idx_workflow (workflow_id),
  INDEX idx_category (category_id)
);
```

---

## 🔗 Associations (Sequelize)

```typescript
// models/workflow-template.model.ts
WorkflowTemplate.belongsToMany(WorkflowCategory, {
  through: 'workflow_category_mappings',
  foreignKey: 'workflow_id',
  otherKey: 'category_id',
  as: 'categories'
});
```

---

*Last Updated: 2026-02-15*
