# document_statuses Table

**Type:** Lookup Table (Metadata)  
**Tenant Isolation:** N/A (Single-Tenant)

---

## 📋 Schema

```sql
CREATE TABLE document_statuses (
  id              VARCHAR(36) PRIMARY KEY,
  
  name            VARCHAR(50) NOT NULL, -- Ví dụ: "Draft", "Review", "Published", "Archived"
  description     TEXT,
  order_index     INT DEFAULT 0,
  
  state_type      ENUM('draft', 'review', 'published', 'archived') NOT NULL,
  
  created_at      TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at      TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  
  INDEX idx_order (order_index)
);
```

---

## 🔗 Associations (Sequelize)

```typescript
// models/document-status.model.ts
DocumentStatus.hasMany(Document, {
  foreignKey: 'status_id',
  as: 'documents'
});
```

---

*Last Updated: 2026-02-15*
