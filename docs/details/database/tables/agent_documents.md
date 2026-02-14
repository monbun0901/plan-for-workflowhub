# agent_documents Table

**Type:** Junction Table (Many-to-Many)  
**Purpose:** Connects AI Agents to specific internal Documents to establish their Knowledge Base (RAG).

---

## 📋 Schema

```sql
CREATE TABLE agent_documents (
  agent_id        VARCHAR(36) NOT NULL REFERENCES agents(id) ON DELETE CASCADE,
  document_id     VARCHAR(36) NOT NULL REFERENCES documents(id) ON DELETE CASCADE,
  
  -- Metadata
  created_at      TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  
  PRIMARY KEY (agent_id, document_id),
  INDEX idx_agent (agent_id),
  INDEX idx_document (document_id)
);
```

---

## 🔗 Associations (Sequelize)

```typescript
// models/agent.model.ts
Agent.belongsToMany(Document, {
  through: 'agent_documents',
  foreignKey: 'agent_id',
  otherKey: 'document_id',
  as: 'knowledge_base'
});

// models/document.model.ts
Document.belongsToMany(Agent, {
  through: 'agent_documents',
  foreignKey: 'document_id',
  otherKey: 'agent_id',
  as: 'agents'
});
```

---

## 📝 Usage Logic

1. **RAG (Retrieval Augmented Generation)**: Khi một Agent được hỏi, hệ thống sẽ tra cứu bảng này để biết Agent đó có quyền đọc những Document nào.
2. **Knowledge Scoping**: Cho phép một Agent chuyên biệt (vd: "Legal Agent") chỉ được tiếp cận các tài liệu pháp lý, tránh bị nhiễu bởi các tài liệu không liên quan.
3. **Cascading**: Nếu xóa 1 Agent hoặc 1 Document, các liên kết trong bảng này sẽ tự động bị xóa theo.

---

*Last Updated: 2026-02-15*
