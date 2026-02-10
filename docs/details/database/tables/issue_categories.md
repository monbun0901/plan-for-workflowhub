# issue_categories Table

**Type:** Junction Table (Issue ↔ Category)  
**Tenant Isolation:** ✅ Required (`organization_id`)

---

## 📋 Schema

```sql
CREATE TABLE issue_categories (
  issue_id        VARCHAR(36) NOT NULL REFERENCES issues(id) ON DELETE CASCADE,
  category_id     VARCHAR(36) NOT NULL REFERENCES categories(id) ON DELETE CASCADE,
  organization_id VARCHAR(36) NOT NULL REFERENCES organizations(id),
  
  PRIMARY KEY (issue_id, category_id),
  INDEX idx_category (category_id),
  INDEX idx_issue (issue_id)
);
```

---

## 🎯 Purpose
Cho phép một Issue được gắn vào nhiều danh mục phân loại.

---

## 🔗 Associations (Sequelize)

```typescript
// models/issue.model.ts
Issue.belongsToMany(Category, {
  through: 'issue_categories',
  foreignKey: 'issue_id',
  otherKey: 'category_id',
  as: 'categories'
});
```

---

*Last Updated: 2026-02-11*
