# task_assignees Table

**Type:** Junction Table (Task ↔ User)  
**Tenant Isolation:** ✅ Required (`organization_id`)

---

## 📋 Schema

```sql
CREATE TABLE task_assignees (
  task_id         VARCHAR(36) NOT NULL REFERENCES tasks(id) ON DELETE CASCADE,
  user_id         VARCHAR(36) NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  organization_id VARCHAR(36) NOT NULL REFERENCES organizations(id),
  
  assigned_at     TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  
  PRIMARY KEY (task_id, user_id),
  INDEX idx_user (user_id),
  INDEX idx_task (task_id)
);
```

---

## 🎯 Purpose
Cho phép một Task được thực hiện bởi một nhóm người (Multi-assignee). 

**Ví dụ:** 
Một task "Viết Unit Test cho Auth Module" có thể được giao cho 2 Developer cùng phối hợp làm.

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

## 🎯 Common Queries

### Get all workers of a task

```typescript
const task = await Task.findByPk(taskId, {
  include: [{ model: User, as: 'assignees' }]
});
```

---

*Last Updated: 2026-02-11*
