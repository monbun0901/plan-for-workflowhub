# project_members Table

**Type:** Junction Table (Project ↔ User)  
**Tenant Isolation:** ✅ Required (`organization_id`)

---

## 📋 Schema

```sql
CREATE TABLE project_members (
  id              VARCHAR(36) PRIMARY KEY,
  project_id      VARCHAR(36) NOT NULL REFERENCES projects(id) ON DELETE CASCADE,
  user_id         VARCHAR(36) NOT NULL REFERENCES users(id),
  organization_id VARCHAR(36) NOT NULL REFERENCES organizations(id),
  
  role            ENUM('lead', 'member', 'viewer') DEFAULT 'member',
  
  joined_at       TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  
  UNIQUE KEY unique_project_user (project_id, user_id),
  INDEX idx_project (project_id),
  INDEX idx_user (user_id)
);
```

---

## 🎯 Purpose
Quản lý "Assignee" ở cấp độ Project. Thay vì Project chỉ có một người quản lý, bảng này cho phép một nhóm người cùng làm việc trong dự án với các vai trò khác nhau.

---

## 🔗 Associations (Sequelize)

```typescript
// models/project-member.model.ts
ProjectMember.belongsTo(Project, { foreignKey: 'project_id', as: 'project' });
ProjectMember.belongsTo(User, { foreignKey: 'user_id', as: 'user' });
```

**Explanation:**
- `lead`: Có quyền quản lý Task/Issue và config project.
- `member`: Có quyền tạo/chỉnh sửa Task/Issue.
- `viewer`: Chỉ có quyền xem.

---

## 🎯 Common Queries

### List all members of a project

```typescript
const members = await ProjectMember.findAll({
  where: { project_id: projectId },
  include: [{ model: User, as: 'user' }]
});
```

### Check if user has access to project

```typescript
const access = await ProjectMember.findOne({
  where: { project_id: projectId, user_id: userId }
});
```

---

*Last Updated: 2026-02-11*
