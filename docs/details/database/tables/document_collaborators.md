# document_collaborators Table

**Type:** Junction Table (Document ↔ User)  
**Tenant Isolation:** N/A (Single-Tenant)

---

## 📋 Schema

```sql
CREATE TABLE document_collaborators (
  document_id     VARCHAR(36) NOT NULL REFERENCES documents(id) ON DELETE CASCADE,
  user_id         VARCHAR(36) NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  
  role            ENUM('viewer', 'editor', 'admin') DEFAULT 'viewer',
  assigned_at     TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  
  PRIMARY KEY (document_id, user_id),
  INDEX idx_user (user_id),
  INDEX idx_doc (document_id)
);
```

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
  as: 'collaborationDocuments'
});
```

---

*Last Updated: 2026-02-15*
