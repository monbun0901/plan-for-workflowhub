# document_category_mappings Table

**Type:** Junction Table (Document ↔ Category)  
**Tenant Isolation:** N/A (Single-Tenant)

---

## 📋 Schema

```sql
CREATE TABLE document_category_mappings (
  document_id     VARCHAR(36) NOT NULL REFERENCES documents(id) ON DELETE CASCADE,
  category_id     VARCHAR(36) NOT NULL REFERENCES document_categories(id) ON DELETE CASCADE,
  
  PRIMARY KEY (document_id, category_id),
  INDEX idx_category (category_id),
  INDEX idx_document (document_id)
);
```

---

## 🔗 Associations (Sequelize)

```typescript
// models/document.model.ts
Document.belongsToMany(DocumentCategory, {
  through: 'document_category_mappings',
  foreignKey: 'document_id',
  otherKey: 'category_id',
  as: 'categories'
});
```

---

*Last Updated: 2026-02-15*
