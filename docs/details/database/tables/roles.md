# roles Table

**Type:** Lookup Table (Identity & Access)  
**Tenant Isolation:** ✅ Required (`organization_id`)

---

## 📋 Schema

```sql
CREATE TABLE roles (
  id              VARCHAR(36) PRIMARY KEY,
  organization_id VARCHAR(36) NOT NULL REFERENCES organizations(id),
  
  name            VARCHAR(50) NOT NULL,          -- 'Admin', 'Developer', 'Viewer'
  description     TEXT,
  
  -- Logic mapping (System roles)
  -- Giúp code nhận diện các quyền tối thượng
  is_system       BOOLEAN DEFAULT FALSE,         -- TRUE cho các role mặc định ko được xóa
  role_type       ENUM('owner', 'admin', 'custom') DEFAULT 'custom',
  
  -- Fine-grained permissions
  permissions     JSON,                          -- ['project:create', 'task:edit', 'member:invite']
  
  created_at      TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at      TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  
  UNIQUE KEY unique_role_name (organization_id, name)
);
```

---

## 🎯 Purpose
Cho phép mỗi tổ chức tự định nghĩa các vai trò và bộ quyền (Capabilities) riêng biệt.

---

## 📝 Permissions Example
```json
{
  "permissions": [
    "org:view",
    "project:create",
    "project:delete",
    "issue:manage",
    "task:assign",
    "billing:manage"
  ]
}
```

---

*Last Updated: 2026-02-11*
