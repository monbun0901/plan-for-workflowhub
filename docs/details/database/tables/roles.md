# roles Table

**Type:** Lookup Table (Identity & Access)  
**Tenant Isolation:** N/A (Global roles for the single organization)

---

## 📋 Schema

```sql
CREATE TABLE roles (
  id              VARCHAR(36) PRIMARY KEY,
  
  name            VARCHAR(50) UNIQUE NOT NULL,    -- 'Admin', 'Manager', 'Employee', 'Viewer'
  description     TEXT,
  
  -- Logic mapping (System roles)
  is_system       BOOLEAN DEFAULT FALSE,         -- TRUE cho các role mặc định ko được xóa
  role_type       ENUM('admin', 'manager', 'user', 'viewer') DEFAULT 'user',
  
  -- Fine-grained permissions
  permissions     JSON,                          -- ['project:create', 'task:edit', 'user:invite']
  
  created_at      TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at      TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);
```

---

## 🎯 Purpose
Định nghĩa các vai trò và bộ quyền (Capabilities) cho toàn hệ thống. Vì đây là Boilerplate Single-Tenant, các role này áp dụng cho mọi người dùng trong hệ thống duy nhất.

---

## 📝 Permissions Example
```json
{
  "permissions": [
    "project:view",
    "project:create",
    "project:delete",
    "issue:manage",
    "task:assign",
    "user:manage"
  ]
}
```

---

*Last Updated: 2026-02-15*
