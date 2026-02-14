# task_statuses Table

**Type:** Lookup Table (Metadata)  
**Tenant Isolation:** N/A (Single-Tenant)

---

## 📋 Schema

```sql
CREATE TABLE task_statuses (
  id              VARCHAR(36) PRIMARY KEY,
  
  name            VARCHAR(50) NOT NULL, -- Ví dụ: "To Do", "In Progress", "Review", "Done"
  description     TEXT,
  order_index     INT DEFAULT 0,
  
  -- Core logic mapping
  state_type      ENUM('todo', 'in_progress', 'done', 'closed') NOT NULL,
  
  created_at      TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at      TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  
  INDEX idx_order (order_index)
);
```

---

## 🔗 Associations (Sequelize)

```typescript
// models/task-status.model.ts
TaskStatus.hasMany(Task, {
  foreignKey: 'status_id',
  as: 'tasks'
});
```

---

*Last Updated: 2026-02-15*
