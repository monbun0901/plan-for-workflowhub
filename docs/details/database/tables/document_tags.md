# document_tags Table

**Type:** Junction Table (Document ↔ Tag)  
**Tenant Isolation:** ✅ Required (`organization_id`) - For fast tenant-scoped scanning.

---

## 📋 Schema

```sql
CREATE TABLE document_tags (
  document_id     VARCHAR(36) NOT NULL REFERENCES documents(id) ON DELETE CASCADE,
  tag_id          VARCHAR(36) NOT NULL REFERENCES tags(id) ON DELETE CASCADE,
  organization_id VARCHAR(36) NOT NULL REFERENCES organizations(id),
  
  PRIMARY KEY (document_id, tag_id),
  INDEX idx_document (document_id),
  INDEX idx_tag (tag_id),
  INDEX idx_org (organization_id)
);
```

---

## 🎯 Purpose
Cho phép gắn nhiều nhãn (Tags) cho một tài liệu. Điều này cực kỳ quan trọng cho hệ thống Knowledge Base để:
1.  **Phân loại chéo:** Một tài liệu có thể vừa là "API" vừa là "v1.0".
2.  **RAG Integration:** Giúp AI lọc tài liệu theo Tag để tìm kiếm ngữ cảnh chính xác hơn.
3.  **Hệ thống tìm kiếm:** Dễ dàng tìm kiếm tất cả tài liệu có tag "Security".

---

## 🔗 Associations (Sequelize)

```typescript
// models/document.model.ts
Document.belongsToMany(Tag, {
  through: 'document_tags',
  foreignKey: 'document_id',
  otherKey: 'tag_id',
  as: 'tags'
});

// models/tag.model.ts
Tag.belongsToMany(Document, {
  through: 'document_tags',
  foreignKey: 'tag_id',
  otherKey: 'document_id',
  as: 'documents'
});
```

---

## 🎯 Common Queries

### Find documents containing ALL specific tags

```typescript
const docs = await Document.findAll({
  include: [{
    model: Tag,
    as: 'tags',
    where: { name: ['Security', 'V1.0'] }
  }],
  group: 'Document.id',
  having: sequelize.literal('COUNT(DISTINCT tags.id) = 2')
});
```

---

## 📚 Related Tables
- **documents** - The target document
- **tags** - Global tag master data

---

*Last Updated: 2026-02-11*
