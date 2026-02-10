# documents Table

**Type:** Knowledge Base  
**Tenant Isolation:** ✅ Required (`organization_id`)

---

## 📋 Schema

```sql
CREATE TABLE documents (
  id              VARCHAR(36) PRIMARY KEY,
  -- Link dự án (Nullable: NULL nếu là tài liệu chung của Organization)
  project_id      VARCHAR(36) REFERENCES projects(id),
  organization_id VARCHAR(36) NOT NULL REFERENCES organizations(id),
  
  title           VARCHAR(200) NOT NULL,
  slug            VARCHAR(200) NOT NULL,
  content         LONGTEXT,                      -- Markdown
  content_type    ENUM('markdown', 'rich_text') DEFAULT 'markdown',
  
  -- Organization & Classification
  -- Category dùng bảng trung gian document_categories để hỗ trợ danh sách
  -- Tags dùng bảng trung gian document_tags
  
  parent_id       VARCHAR(36) REFERENCES documents(id),
  order_index     INT DEFAULT 0,
  
  status          ENUM('draft', 'published', 'archived') DEFAULT 'draft',
  visibility_id   VARCHAR(36) REFERENCES visibility_levels(id),
  version         INT DEFAULT 1,
  
  created_by      VARCHAR(36) NOT NULL REFERENCES users(id),
  updated_by      VARCHAR(36) REFERENCES users(id),
  
  -- Vector embedding for RAG
  embedding_status ENUM('pending', 'processing', 'completed', 'failed') DEFAULT 'pending',
  last_embedded_at TIMESTAMP,
  
  created_at      TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at      TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  
  UNIQUE KEY unique_project_slug (project_id, slug),
  INDEX idx_org (organization_id),
  INDEX idx_category (organization_id, category),
  INDEX idx_embedding (organization_id, embedding_status)
);
```

---

## 🔗 Associations (Sequelize)

```typescript
// models/document.model.ts
Document.belongsTo(Project, {
  foreignKey: 'project_id',
  as: 'project'
});

Document.belongsTo(Document, {
  foreignKey: 'parent_id',
  as: 'parent'
});

Document.hasMany(Document, {
  foreignKey: 'parent_id',
  as: 'children'
});

Document.belongsToMany(Category, {
  through: 'document_categories',
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

Document.belongsTo(User, {
  foreignKey: 'created_by',
  as: 'creator'
});

Document.belongsTo(User, {
  foreignKey: 'updated_by',
  as: 'updater'
});

Document.belongsToMany(Tag, {
  through: 'document_tags',
  foreignKey: 'document_id',
  otherKey: 'tag_id',
  as: 'tags'
});
```

**Explanation:**
- `belongsTo(Project)` - Document thuộc 1 project
- `belongsTo(Document, parent)` - Self-referencing (nested docs)
- `hasMany(Document, children)` - Hierarchy structure
- `belongsTo(User, creator/updater)` - Track authorship

---

## 📝 Fields Explanation - Classification

| Field | Description | Example |
|-------|-------------|---------|
| `category` | Document type | 'api', 'architecture', 'tutorial', 'internal' |
| `tags` | Flexible tags | `['v1.0', 'backend', 'security', 'deprecated']` |
| `version` | Doc version number | 1, 2, 3... |

**Category Examples:**
- `api` - API documentation
- `architecture` - Architecture decisions, design docs
- `tutorial` - How-to guides, tutorials
- `reference` - Reference materials
- `internal` - Internal company docs
- `changelog` - Version changelogs

**Tags Usage:**
- **Version tags**: `v1.0`, `v2.0`, `v3.0`
- **Tech stack**: `nodejs`, `react`, `mysql`
- **Team/module**: `backend`, `frontend`, `devops`
- **Status**: `deprecated`, `wip`, `reviewed`

### RAG Integration

| Field | Description | RAG Usage |
|-------|-------------|-----------|
| `content` | Markdown text | Chunked for embeddings |
| `embedding_status` | Processing state | Track vector creation |
| `last_embedded_at` | Last sync time | Know if stale |

**RAG Flow:**
1. Document created → `embedding_status: 'pending'`
2. Background job → chunks content → generate embeddings
3. Store in Vector DB with `organization_id` + `category` + `tags` filter
4. Update `embedding_status: 'completed'`

---

## 🎯 Common Queries

### Get documents by category (Multi-select)

```typescript
const docs = await Document.findAll({
  where: {
    organization_id: orgId,
    status: 'published'
  },
  include: [{
    model: Category,
    as: 'categories',
    where: { id: [catId1, catId2] }
  }],
  order: [['version', 'DESC']]
});
```

### Get documents with collaborators

```typescript
const doc = await Document.findByPk(docId, {
  include: [{ 
    model: User, 
    as: 'collaborators',
    through: { attributes: ['role'] } // Get collaborator role
  }]
});
```

### Search by tags (JSON contains)

```typescript
// Find all v1.0 backend docs
const docs = await Document.findAll({
  where: {
    organization_id: orgId,
    tags: {
      [Op.contains]: ['v1.0', 'backend']
    },
    status: 'published'
  }
});
```

### Get latest version of each doc

```typescript
const latestDocs = await sequelize.query(`
  SELECT d.*
  FROM documents d
  INNER JOIN (
    SELECT slug, MAX(version) as max_version
    FROM documents
    WHERE organization_id = :orgId AND project_id = :projectId
    GROUP BY slug
  ) latest ON d.slug = latest.slug AND d.version = latest.max_version
  WHERE d.organization_id = :orgId
`, {
  replacements: { orgId, projectId },
  type: QueryTypes.SELECT,
  model: Document
});
```

### Get docs ready for embedding

```typescript
const docs = await Document.findAll({
  where: {
    organization_id: orgId,
    embedding_status: 'pending',
    status: 'published'
  }
});
```

### Nested document tree

```typescript
const rootDocs = await Document.findAll({
  where: {
    project_id: projectId,
    organization_id: orgId,
    parent_id: null
  },
  include: [{
    model: Document,
    as: 'children',
    hierarchy: true  // Recursive
  }]
});
```

---

## 🔒 Tenant Isolation in Vector DB

```typescript
// When creating embeddings
await vectorDB.add({
  id: document.id,
  content: chunkText,
  metadata: {
    organization_id: document.organization_id,  // CRITICAL
    project_id: document.project_id,
    document_id: document.id
  }
});

// When querying (RAG)
const results = await vectorDB.query({
  vector: queryEmbedding,
  filter: {
    organization_id: { $eq: orgId }  // CRITICAL
  }
});
```

---

## 📚 Related Tables

- **projects** - Parent project
- **document_versions** - Version history (separate table)
- **Vector DB** - External (Chroma/Pinecone)

---

*Last Updated: 2026-02-11*
