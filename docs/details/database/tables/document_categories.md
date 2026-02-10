# document_categories Table

**Type:** Junction Table (Document ↔ Category)  
**Tenant Isolation:** ✅ Required (`organization_id`)

---

## 📋 Schema

```sql
CREATE TABLE document_categories (
  document_id     VARCHAR(36) NOT NULL REFERENCES documents(id) ON DELETE CASCADE,
  category_id     VARCHAR(36) NOT NULL REFERENCES categories(id) ON DELETE CASCADE,
  organization_id VARCHAR(36) NOT NULL REFERENCES organizations(id),
  
  PRIMARY KEY (document_id, category_id),
  INDEX idx_category (category_id),
  INDEX idx_document (document_id)
);
```

---

## 🎯 Purpose
Cho phép một tài liệu (Document) thuộc về nhiều danh mục khác nhau. Phù hợp cho việc tổ chức Knowledge Base đa chiều.

**Ví dụ:**
Một tài liệu "Security Best Practices" có thể thuộc cả 2 danh mục: **"DevOps"** và **"Security"**.

---

## 🔗 Associations (Sequelize)

```typescript
// models/document.model.ts
Document.belongsToMany(Category, {
  through: 'document_categories',
  foreignKey: 'document_id',
  otherKey: 'category_id',
  as: 'categories'
});

// models/category.model.ts
Category.belongsToMany(Document, {
  through: 'document_categories',
  foreignKey: 'category_id',
  otherKey: 'document_id',
  as: 'documents'
});
```

---

*Last Updated: 2026-02-11*
