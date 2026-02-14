# documents Table

**Type:** Knowledge Base  
**Tenant Isolation:** N/A (Single-Tenant)

---

## 📋 Schema

```sql
CREATE TABLE documents (
  id              VARCHAR(36) PRIMARY KEY,
  project_id      VARCHAR(36) REFERENCES projects(id),
  
  title           VARCHAR(200) NOT NULL,
  slug            VARCHAR(200) NOT NULL,
  content         LONGTEXT,                      -- Markdown
  content_type    ENUM('markdown', 'rich_text') DEFAULT 'markdown',
  
  parent_id       VARCHAR(36) REFERENCES documents(id),
  order_index     INT DEFAULT 0,
  
  status_id       VARCHAR(36) REFERENCES document_statuses(id), -- Default: "Draft" (Init by App)
  visibility      ENUM('public', 'private', 'restricted') DEFAULT 'public',
  version         VARCHAR(50) DEFAULT '1.0.0',
  
  created_by      VARCHAR(36) NOT NULL REFERENCES users(id),
  updated_by      VARCHAR(36) REFERENCES users(id),
  
  -- Vector embedding for RAG
  embedding_status ENUM('pending', 'processing', 'completed', 'failed') DEFAULT 'pending',
  last_embedded_at TIMESTAMP,
  
  created_at      TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at      TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  
  UNIQUE KEY unique_project_slug (project_id, slug),
  INDEX idx_status (status_id),
  INDEX idx_embedding (embedding_status)
);
```

---

## 🔗 Associations (Sequelize)

```typescript
// models/document.model.ts
Document.belongsTo(Project, { foreignKey: 'project_id', as: 'project' });
Document.belongsTo(Document, { foreignKey: 'parent_id', as: 'parent' });
Document.hasMany(Document, { foreignKey: 'parent_id', as: 'children' });

Document.belongsToMany(DocumentCategory, {
  through: 'document_category_mappings',
  foreignKey: 'document_id',
  otherKey: 'category_id',
  as: 'categories'
});

Document.belongsToMany(User, {
  through: 'document_collaborators',
  foreignKey: 'document_id',
  otherKey: 'user_id',
  as: 'collaborators'
});

Document.belongsTo(User, { foreignKey: 'created_by', as: 'creator' });
Document.belongsTo(User, { foreignKey: 'updated_by', as: 'updater' });
```

---

## 📝 RAG Integration

**RAG Flow:**
1. Document created → `embedding_status: 'pending'`
2. Background job → chunks content → generate embeddings
3. Store in Vector DB with `document_id` + `project_id` metadata.
4. Update `embedding_status: 'completed'`

---

## 🎯 Common Queries

### Get all published documents in a project

```typescript
const docs = await Document.findAll({
  where: { project_id: projectId },
  include: [
    { model: DocumentStatus, as: 'status' },
    { model: DocumentCategory, as: 'categories' }
  ],
  order: [['order_index', 'ASC']]
});
```

---

*Last Updated: 2026-02-15*
