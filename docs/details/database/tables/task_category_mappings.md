# task_category_mappings Table

**Type:** Junction Table (Task ↔ Category)  
**Tenant Isolation:** N/A (Single-Tenant)

---

## 📋 Schema

```sql
CREATE TABLE task_category_mappings (
  task_id         VARCHAR(36) NOT NULL REFERENCES tasks(id) ON DELETE CASCADE,
  category_id     VARCHAR(36) NOT NULL REFERENCES task_categories(id) ON DELETE CASCADE,
  
  PRIMARY KEY (task_id, category_id),
  INDEX idx_task (task_id),
  INDEX idx_category (category_id)
);
```

---

## 🔗 Associations (Sequelize)

```typescript
// models/task.model.ts
Task.belongsToMany(TaskCategory, {
  through: 'task_category_mappings',
  foreignKey: 'task_id',
  otherKey: 'category_id',
  as: 'categories'
});
```

---

*Last Updated: 2026-02-15*
