# issue_category_mappings Table

**Type:** Junction Table (Issue ↔ Category)  
**Tenant Isolation:** N/A (Single-Tenant)

---

## 📋 Schema

```sql
CREATE TABLE issue_category_mappings (
  issue_id        VARCHAR(36) NOT NULL REFERENCES issues(id) ON DELETE CASCADE,
  category_id     VARCHAR(36) NOT NULL REFERENCES issue_categories(id) ON DELETE CASCADE,
  
  PRIMARY KEY (issue_id, category_id),
  INDEX idx_category (category_id),
  INDEX idx_issue (issue_id)
);
```

---

## 🔗 Associations (Sequelize)

```typescript
// models/issue.model.ts
Issue.belongsToMany(IssueCategory, {
  through: 'issue_category_mappings',
  foreignKey: 'issue_id',
  otherKey: 'category_id',
  as: 'categories'
});
```

---

*Last Updated: 2026-02-15*
