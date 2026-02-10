# task_categories Table

**Type:** Junction Table (Task ↔ Category)  
**Tenant Isolation:** ✅ Required (`organization_id`)

---

## 📋 Schema

```sql
CREATE TABLE task_categories (
  task_id         VARCHAR(36) NOT NULL REFERENCES tasks(id) ON DELETE CASCADE,
  category_id     VARCHAR(36) NOT NULL REFERENCES categories(id) ON DELETE CASCADE,
  organization_id VARCHAR(36) NOT NULL REFERENCES organizations(id),
  
  PRIMARY KEY (task_id, category_id),
  INDEX idx_category (category_id),
  INDEX idx_task (task_id)
);
```

---

## 🎯 Purpose
Cho phép một Task thuộc về nhiều danh mục khác nhau (Multi-categorization).

**Ví dụ:**
Một Task "Fix CSS Bug" có thể thuộc cả 2 category: **"Frontend"** và **"Urgent Fix"**.

---

## 🔗 Associations (Sequelize)

```typescript
// models/task.model.ts
Task.belongsToMany(Category, {
  through: 'task_categories',
  foreignKey: 'task_id',
  otherKey: 'category_id',
  as: 'categories'
});

// models/category.model.ts
Category.belongsToMany(Task, {
  through: 'task_categories',
  foreignKey: 'category_id',
  otherKey: 'task_id',
  as: 'tasks'
});
```

---

## 🎯 Common Queries

### Find all tasks with specific categories (In query)

```typescript
const tasks = await Task.findAll({
  include: [{
    model: Category,
    as: 'categories',
    where: { id: [catId1, catId2] }
  }]
});
```

---

*Last Updated: 2026-02-11*
