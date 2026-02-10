# categories Table

**Type:** Lookup Table (Metadata)  
**Tenant Isolation:** ✅ Required (`organization_id`)

---

## 📋 Schema

```sql
CREATE TABLE categories (
  id              VARCHAR(36) PRIMARY KEY,
  organization_id VARCHAR(36) NOT NULL REFERENCES organizations(id),
  
  name            VARCHAR(100) NOT NULL,
  description     TEXT,
  
  -- Target entity this category applies to
  target_type     ENUM('project', 'issue', 'task', 'document') NOT NULL,
  
  created_at      TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at      TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  
  INDEX idx_org_target (organization_id, target_type)
);
```

---

## 🎯 Purpose
Cho phép mỗi tổ chức tự định nghĩa các phân loại (Category/Department) cho dữ liệu của họ thay vì dùng giá trị cứng.

**Ví dụ:**
- **Project Categories:** "Client Projects", "Internal R&D", "Marketing Campaigns".
- **Task Categories:** "Frontend", "Backend", "Design", "DevOps".

---

## 🔗 Associations (Sequelize)

```typescript
// models/category.model.ts
Category.belongsTo(Organization, {
  foreignKey: 'organization_id',
  as: 'organization'
});

Category.hasMany(Project, { foreignKey: 'category_id', as: 'projects' });

Category.belongsToMany(Issue, {
  through: 'issue_categories',
  foreignKey: 'category_id',
  otherKey: 'issue_id',
  as: 'issues'
});

Category.belongsToMany(Task, {
  through: 'task_categories',
  foreignKey: 'category_id',
  otherKey: 'task_id',
  as: 'tasks'
});
```

---

## 🎯 Common Queries

### List categories for Tasks in an Org

```typescript
const taskCategories = await Category.findAll({
  where: {
    organization_id: orgId,
    target_type: 'task'
  }
});
```

---

## ✅ Best Practices
- **Seed Defaults:** Khi tạo Organization mới, hãy chuyển một bộ category mặc định vào bảng này.

---

*Last Updated: 2026-02-11*
