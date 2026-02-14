# member_category_mappings Table

**Type:** Junction Table (User ↔ Category)  
**Tenant Isolation:** N/A (Single-Tenant)

---

## 📋 Schema

```sql
CREATE TABLE member_category_mappings (
  user_id         VARCHAR(36) NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  category_id     VARCHAR(36) NOT NULL REFERENCES member_categories(id) ON DELETE CASCADE,
  
  PRIMARY KEY (user_id, category_id),
  INDEX idx_user (user_id),
  INDEX idx_category (category_id)
);
```

---

## 🔗 Associations (Sequelize)

```typescript
// models/user.model.ts
User.belongsToMany(MemberCategory, {
  through: 'member_category_mappings',
  foreignKey: 'user_id',
  otherKey: 'category_id',
  as: 'categories'
});
```

---

*Last Updated: 2026-02-15*
