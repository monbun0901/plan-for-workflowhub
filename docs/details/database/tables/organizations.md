# organizations Table

**Type:** Core Multi-Tenant  
**Tenant Isolation:** N/A (this is the tenant root)

---

## 📋 Schema

```sql
CREATE TABLE organizations (
  id            VARCHAR(36) PRIMARY KEY,
  name          VARCHAR(100) NOT NULL,
  slug          VARCHAR(100) UNIQUE NOT NULL,   -- URL-friendly identifier
  description   TEXT,
  logo_url      VARCHAR(500),
  settings      JSON,                           -- Org-level settings
  created_by    VARCHAR(36) REFERENCES users(id),
  created_at    TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at    TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);
```

---

## 🔑 Indexes

```sql
PRIMARY KEY (id)
UNIQUE KEY unique_slug (slug)
CREATE INDEX idx_orgs_slug ON organizations(slug);
CREATE INDEX idx_orgs_created_by ON organizations(created_by);
```

---

## 🔗 Associations (Sequelize)

```typescript
// models/organization.model.ts
Organization.hasMany(Member, {
  foreignKey: 'organization_id',
  as: 'members'
});

Organization.hasMany(Project, {
  foreignKey: 'organization_id',
  as: 'projects'
});

Organization.belongsTo(User, {
  foreignKey: 'created_by',
  as: 'creator'
});
```

**Explanation:**
- `hasMany(Member)` - 1 organization có nhiều members (users join vào org)
- `hasMany(Project)` - 1 organization có nhiều projects
- `belongsTo(User)` - Organization được tạo bởi 1 user (founder)

---

## 📝 Fields Explanation

| Field | Type | Description | Required |
|-------|------|-------------|----------|
| `id` | UUID | Primary key | ✅ |
| `name` | VARCHAR(100) | Organization name | ✅ |
| `slug` | VARCHAR(100) | URL slug (e.g., "acme-corp") | ✅ |
| `description` | TEXT | Organization description | ❌ |
| `logo_url` | VARCHAR(500) | Logo image URL | ❌ |
| `settings` | JSON | Org preferences (theme, cấu hình chung) | ❌ |
| `created_by` | UUID | Founder user ID | ✅ |

### settings JSON Structure (Ví dụ)
Dùng để lưu các cấu hình linh hoạt mà không muốn tạo cột riêng:
- `default_timezone`: Múi giờ mặc định cho toàn Org.
- `allow_guest_access`: Cho phép truy cập không cần login (Public view).
- `ai_config`: Các tham số giới hạn cho AI Agents.
- `theme`: Các cấu hình giao diện đặc thù (nếu có).

---

## 🎯 Common Queries

### Get org with members

```typescript
const org = await Organization.findByPk(orgId, {
  include: [{
    model: Member,
    as: 'members',
    where: { status: 'active' },
    include: [{ model: User, as: 'user' }]
  }]
});
```

### Find org by slug (for public URLs)

```typescript
const org = await Organization.findOne({
  where: { slug: 'acme-corp' }
});
```

---

## ✅ Validation Rules

- `name`: 1-100 characters
- `slug`: lowercase, alphanumeric + hyphens only, unique
- `settings`: valid JSON object

---

## 📚 Related Tables

- **members** - User memberships
- **projects** - Projects in this org
- **issues, tasks, documents, etc.** - All scoped to org

---

*Last Updated: 2026-02-11*
