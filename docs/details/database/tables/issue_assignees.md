# issue_assignees Table

**Type:** Junction Table (Issue ↔ User)  
**Tenant Isolation:** ✅ Required (`organization_id`)

---

## 📋 Schema

```sql
CREATE TABLE issue_assignees (
  issue_id        VARCHAR(36) NOT NULL REFERENCES issues(id) ON DELETE CASCADE,
  user_id         VARCHAR(36) NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  organization_id VARCHAR(36) NOT NULL REFERENCES organizations(id),
  
  assigned_at     TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  
  PRIMARY KEY (issue_id, user_id),
  INDEX idx_user (user_id),
  INDEX idx_issue (issue_id)
);
```

---

## 🎯 Purpose
Cho phép một Issue được xử lý bởi một nhóm người (Multi-assignee). Phù hợp cho các vấn đề phức tạp cần sự phối hợp giữa nhiều bộ phận.

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

## 🎯 Common Queries

### Get all assignees of an issue

```typescript
const issue = await Issue.findByPk(issueId, {
  include: [{ model: User, as: 'assignees' }]
});
```

---

*Last Updated: 2026-02-11*
