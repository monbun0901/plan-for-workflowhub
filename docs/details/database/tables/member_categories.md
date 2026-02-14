# member_categories Table

**Type:** Master Table (Metadata)  
**Tenant Isolation:** N/A (Single-Tenant)

---

## 📋 Schema

```sql
CREATE TABLE member_categories (
  id              VARCHAR(36) PRIMARY KEY,
  
  name            VARCHAR(100) NOT NULL, -- Ví dụ: "Quản lý", "Kỹ thuật", "HR"
  description     TEXT,
  order_index     INT DEFAULT 0,
  
  created_at      TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at      TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);
```

---

## 🔗 Associations (Sequelize)

```typescript
// models/member-category.model.ts
MemberCategory.belongsToMany(User, {
  through: 'member_category_mappings',
  foreignKey: 'category_id',
  otherKey: 'user_id',
  as: 'users'
});
```

---

*Last Updated: 2026-02-15*
