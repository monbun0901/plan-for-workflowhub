# task_assignees Table

**Type:** Junction Table (Task ↔ User)  
**Tenant Isolation:** N/A (Single-Tenant)

---

## 📋 Schema

```sql
CREATE TABLE task_assignees (
  task_id         VARCHAR(36) NOT NULL REFERENCES tasks(id) ON DELETE CASCADE,
  user_id         VARCHAR(36) NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  
  assigned_at     TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  
  PRIMARY KEY (task_id, user_id),
  INDEX idx_user (user_id),
  INDEX idx_task (task_id)
);
```

---

## 🔗 Associations (Sequelize)

```typescript
// models/task.model.ts
Task.belongsToMany(User, {
  through: 'task_assignees',
  foreignKey: 'task_id',
  otherKey: 'user_id',
  as: 'assignees'
});

// models/user.model.ts
User.belongsToMany(Task, {
  through: 'task_assignees',
  foreignKey: 'user_id',
  otherKey: 'task_id',
  as: 'tasks'
});
```

---

*Last Updated: 2026-02-15*
