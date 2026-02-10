# members Table

**Type:** Junction Table (User ↔ Organization)  
**Tenant Isolation:** N/A (bridges users and orgs)

---

## 📋 Schema

```sql
CREATE TABLE members (
  id              VARCHAR(36) PRIMARY KEY,
  user_id         VARCHAR(36) NOT NULL REFERENCES users(id),
  organization_id VARCHAR(36) NOT NULL REFERENCES organizations(id),
  
  -- Dynamic Role
  role_id         VARCHAR(36) REFERENCES roles(id),
  status          ENUM('active', 'pending', 'removed') DEFAULT 'pending',
  invited_by      VARCHAR(36) REFERENCES users(id),
  joined_at       TIMESTAMP,
  created_at      TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  
  UNIQUE KEY unique_user_org (user_id, organization_id)
);
```

---

## 🔑 Indexes

```sql
PRIMARY KEY (id)
UNIQUE KEY unique_user_org (user_id, organization_id)
CREATE INDEX idx_members_user ON members(user_id);
CREATE INDEX idx_members_org ON members(organization_id);
CREATE INDEX idx_members_org_role ON members(organization_id, role);
```

---

## 🔗 Associations (Sequelize)

```typescript
// models/member.model.ts
Member.belongsTo(User, {
  foreignKey: 'user_id',
  as: 'user'
});

Member.belongsTo(Organization, {
  foreignKey: 'organization_id',
  as: 'organization'
});

Member.belongsTo(Role, {
  foreignKey: 'role_id',
  as: 'role'
});

Member.belongsTo(User, {
  foreignKey: 'invited_by',
  as: 'inviter'
});
```

**Explanation:**
- `belongsTo(User)` - Member đại diện cho 1 User cụ thể.
- `belongsTo(Organization)` - Member thuộc 1 tổ chức.
- `belongsTo(Role)` - Vai trò động của member (với các permissions đi kèm).
- `belongsTo(User, inviter)` - Người mời user này vào tổ chức.

---

## 📝 Fields Explanation

| Field | Type | Description |
|-------|------|-------------|
| `user_id` | UUID | User being added to org |
| `organization_id` | UUID | Target organization |
| `role_id` | UUID | Link to `roles` table (Dynamic permissions) |
| `status` | ENUM | active (accepted), pending (invited), removed |

---

## 🎯 Common Queries

### Get member with Role & Permissions

```typescript
const member = await Member.findOne({
  where: { user_id: userId, organization_id: orgId },
  include: [{ 
    model: Role, 
    as: 'role',
    attributes: ['name', 'permissions']
  }]
});

// Check permission
const canCreateProject = member.role.permissions.includes('project:create');
```

---

## 🔒 Security Rules

- `owner` role can only be assigned to creator (1 per org)
- `admin` can invite members and manage projects
- `member` can view and contribute to projects

---

## 📚 Related Tables

- **users** - Member user details
- **organizations** - Organization details

---

*Last Updated: 2026-02-11*
