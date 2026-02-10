# tags Table

**Type:** Lookup Table (Metadata)  
**Tenant Isolation:** ✅ Required (`organization_id`)

---

## 📋 Schema

```sql
CREATE TABLE tags (
  id              VARCHAR(36) PRIMARY KEY,
  organization_id VARCHAR(36) NOT NULL REFERENCES organizations(id),
  
  name            VARCHAR(50) NOT NULL,
  
  -- Phân loại tag này dùng cho entity nào (Optional - có thể dùng chung)
  target_entity   ENUM('project', 'issue', 'task', 'document', 'all') DEFAULT 'all',
  
  created_at      TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at      TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  
  UNIQUE KEY unique_tag_name (organization_id, name)
);
```

---

## 🎯 Purpose
Quản lý nhãn dán tập trung. Khác với Category (phân loại chính), Tags dùng để đánh dấu các đặc tính phụ, trạng thái tạm thời hoặc các từ khóa tìm kiếm.

---

## 🔗 Associations (Sequelize)

```typescript
// Quan hệ Many-to-Many với các entity khác thông qua bảng junction
Tag.belongsToMany(Project, { through: 'project_tags', foreignKey: 'tag_id' });
Tag.belongsToMany(Issue, { through: 'issue_tags', foreignKey: 'tag_id' });
Tag.belongsToMany(Task, { through: 'task_tags', foreignKey: 'tag_id' });
Tag.belongsToMany(Document, { through: 'document_tags', foreignKey: 'tag_id' });
```

---

*Last Updated: 2026-02-11*
