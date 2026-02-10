# document_collaborators Table

**Type:** Junction Table (Document ↔ User)  
**Tenant Isolation:** ✅ Required (`organization_id`)

---

## 📋 Schema

```sql
CREATE TABLE document_collaborators (
  document_id     VARCHAR(36) NOT NULL REFERENCES documents(id) ON DELETE CASCADE,
  user_id         VARCHAR(36) NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  organization_id VARCHAR(36) NOT NULL REFERENCES organizations(id),
  
  role            ENUM('editor', 'viewer') DEFAULT 'viewer',
  invited_at      TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  
  PRIMARY KEY (document_id, user_id),
  INDEX idx_user (user_id),
  INDEX idx_document (document_id)
);
```

---

## 🎯 Purpose
Quản lý quyền truy cập và những người cùng tham gia biên soạn/theo dõi tài liệu.

**Lưu ý:** `created_by` trong bảng `documents` vẫn giữ vai trò là "Owner" (Chủ sở hữu).

---

## 🔗 Associations (Sequelize)

```typescript
// models/document.model.ts
Document.belongsToMany(User, {
  through: 'document_collaborators',
  foreignKey: 'document_id',
  otherKey: 'user_id',
  as: 'collaborators'
});

// models/user.model.ts
User.belongsToMany(Document, {
  through: 'document_collaborators',
  foreignKey: 'user_id',
  otherKey: 'document_id',
  as: 'sharedDocuments'
});
```

---

*Last Updated: 2026-02-11*
