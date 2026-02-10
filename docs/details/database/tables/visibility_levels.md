# visibility_levels Table

**Type:** Lookup Table (Access Control)  
**Tenant Isolation:** ✅ Required (`organization_id`)

---

## 📋 Schema

```sql
CREATE TABLE visibility_levels (
  id              VARCHAR(36) PRIMARY KEY,
  organization_id VARCHAR(36) NOT NULL REFERENCES organizations(id),
  
  name            VARCHAR(50) NOT NULL,          -- 'Private', 'Internal', 'Public', 'Confidential'
  description     TEXT,
  
  -- Key để code xử lý logic đặc biệt
  level_key       ENUM('private', 'internal', 'public', 'restricted') NOT NULL,
  created_at      TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at      TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  
  UNIQUE KEY unique_visibility_name (organization_id, name)
);
```

---

## 🎯 Purpose
Định nghĩa các mức độ truy cập dữ liệu. Việc dùng bảng giúp PM/Admin có thể tùy chỉnh tên gọi hoặc thêm các mức độ bảo mật mới mà không cần can thiệp vào code.

**Ví dụ:**
- **Private:** Chỉ những người được Assign/Member mới thấy.
- **Internal:** Toàn bộ thành viên trong Organization đều thấy.
- **Public:** Công khai ra ngoài (nếu hệ thống hỗ trợ Guest access).
- **Top Secret:** Chỉ dành cho bộ phận cấp cao.

---

*Last Updated: 2026-02-11*
