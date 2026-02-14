# issue_assignees Table

**Type:** Junction Table (Issue ↔ User)  
**Tenant Isolation:** N/A (Single-Tenant)

---

## 📋 Schema

```sql
CREATE TABLE issue_assignees (
  issue_id        VARCHAR(36) NOT NULL REFERENCES issues(id) ON DELETE CASCADE,
  user_id         VARCHAR(36) NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  
  assigned_at     TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  
  PRIMARY KEY (issue_id, user_id),
  INDEX idx_user (user_id),
  INDEX idx_issue (issue_id)
);
```

---

## 🔗 Associations (Sequelize)

```typescript
// models/issue.model.ts
Issue.belongsToMany(User, {
  through: 'issue_assignees',
  foreignKey: 'issue_id',
  otherKey: 'user_id',
  as: 'assignees'
});

// models/user.model.ts
User.belongsToMany(Issue, {
  through: 'issue_assignees',
  foreignKey: 'user_id',
  otherKey: 'issue_id',
  as: 'assignedIssues'
});
```

---

*Last Updated: 2026-02-15*
