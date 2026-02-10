# task_tags Table

**Type:** Junction Table (Task ↔ Tag)  
**Tenant Isolation:** ✅ Required (`organization_id`)

---

## 📋 Schema

```sql
CREATE TABLE task_tags (
  task_id         VARCHAR(36) NOT NULL REFERENCES tasks(id) ON DELETE CASCADE,
  tag_id          VARCHAR(36) NOT NULL REFERENCES tags(id) ON DELETE CASCADE,
  organization_id VARCHAR(36) NOT NULL REFERENCES organizations(id),
  
  PRIMARY KEY (task_id, tag_id),
  INDEX idx_task (task_id),
  INDEX idx_tag (tag_id)
);
```

---

## 🎯 Purpose
Liên kết Tasks với danh sách nhãn dán tập trung (thay thế cho labels JSON cũ).

---

## 🔗 Associations (Sequelize)

```typescript
// models/task.model.ts
Task.belongsToMany(Tag, {
  through: 'task_tags',
  foreignKey: 'task_id',
  otherKey: 'tag_id',
  as: 'tags'
});
```

---

*Last Updated: 2026-02-11*
