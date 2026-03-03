# agents Table

**Type:** AI Agent Registry  
**Tenant Isolation:** N/A (Single-Tenant)

---

## 📋 Schema

```sql
CREATE TABLE agents (
  id              VARCHAR(36) PRIMARY KEY,
  name            VARCHAR(100) NOT NULL,
  description     VARCHAR(500),
  
  -- ── Role & Identity ──
  role            ENUM('pm', 'developer', 'reviewer', 'analyst', 'writer', 'custom') NOT NULL DEFAULT 'custom',
  avatar_url      VARCHAR(255),
  
  -- ── LLM Configuration ──
  provider        VARCHAR(50) NOT NULL,       -- 'ollama', 'openai', 'anthropic', 'google', 'vllm'
  base_url        VARCHAR(255),               -- Custom endpoint (null = default provider URL)
  model           VARCHAR(100) NOT NULL,      -- 'llama3:8b', 'gpt-4o', 'claude-3.5-sonnet', 'gemini-2.0-flash'
  
  -- ── Prompt Engineering ──
  system_prompt   TEXT NOT NULL,              -- Hướng dẫn hành vi cho Agent
  
  -- ── Flexible Configuration (JSON) ──
  ai_settings     JSON,                       -- { temperature, max_tokens, top_p }
  rag_config      JSON,                       -- { enabled, top_k, similarity_threshold }
  enabled_tools   JSON,                       -- ["create_task", "search_wiki", "update_issue_status"]
  
  -- ── Scope ──
  project_id      VARCHAR(36) REFERENCES projects(id) ON DELETE SET NULL,
  
  -- ── Status & Meta ──
  status          ENUM('active', 'disabled') DEFAULT 'active',
  
  -- ── Usage Stats ──
  total_executions    INT DEFAULT 0,
  total_tokens_used   BIGINT DEFAULT 0,
  last_used_at        TIMESTAMP NULL,
  
  created_at      TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at      TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  
  INDEX idx_status (status),
  INDEX idx_role (role),
  INDEX idx_project (project_id)
);
```

---

## 🔗 Knowledge Base (Spec Hub)

Agent truy xuất tri thức nội bộ thông qua bảng trung gian **[agent_documents](./agent_documents.md)**.

Cho phép:
- **Knowledge Scoping:** Agent chuyên biệt chỉ truy cập tài liệu liên quan
- **RAG Query:** Vector search trên documents đã gắn

---

## 🔗 Associations (Sequelize)

```typescript
// models/agent.model.ts

// Knowledge Base (Spec Hub)
Agent.belongsToMany(Document, {
  through: 'agent_documents',
  foreignKey: 'agent_id',
  otherKey: 'document_id',
  as: 'knowledge_base'
});

// Scoped to Project
Agent.belongsTo(Project, { foreignKey: 'project_id', as: 'project' });

// Generated content
Agent.hasMany(Task, { foreignKey: 'generated_by_agent', as: 'generatedTasks' });
Agent.hasMany(Chat, { foreignKey: 'agent_id', as: 'chats' });
```

---

## 🎯 Relationships

| Relation | Table | Type | Purpose |
|---|---|---|---|
| knowledge_base | agent_documents → documents | M:N | Spec Hub — tài liệu Agent được phép truy cập |
| project | projects | M:1 | Agent scoped cho project cụ thể (optional) |
| chats | chats | 1:N | Các phiên chat của Agent |
| generatedTasks | tasks | 1:N | Tasks được Agent tạo ra |
| workflow_steps | workflow_templates.steps_config | JSON ref | Agent tham gia workflow nào |

---

*Last Updated: 2026-02-25*
