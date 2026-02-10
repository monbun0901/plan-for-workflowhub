# issue_tags Table

**Type:** Junction Table (Issue ↔ Tag)  
**Tenant Isolation:** ✅ Required (`organization_id`)

---

## 📋 Schema

```sql
CREATE TABLE issue_tags (
  issue_id        VARCHAR(36) NOT NULL REFERENCES issues(id) ON DELETE CASCADE,
  tag_id          VARCHAR(36) NOT NULL REFERENCES tags(id) ON DELETE CASCADE,
  organization_id VARCHAR(36) NOT NULL REFERENCES organizations(id),
  
  PRIMARY KEY (issue_id, tag_id),
  INDEX idx_issue (issue_id),
  INDEX idx_tag (tag_id)
);
```

---

## 🎯 Purpose
Liên kết Issues với danh sách nhãn dán tập trung (thay thế cho labels JSON cũ).

---

## 🔗 Associations (Sequelize)

```typescript
// models/issue.model.ts
Issue.belongsToMany(Tag, {
  through: 'issue_tags',
  foreignKey: 'issue_id',
  otherKey: 'tag_id',
  as: 'tags'
});
```

---

*Last Updated: 2026-02-11*
