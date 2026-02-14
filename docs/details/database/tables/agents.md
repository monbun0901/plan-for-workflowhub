# agents Table

**Type:** AI Agent Registry  
**Tenant Isolation:** N/A (Single-Tenant)

---

## 📋 Schema

```sql
CREATE TABLE agents (
  id              VARCHAR(36) PRIMARY KEY,
  name            VARCHAR(100) NOT NULL,
  description     TEXT,
  
  -- LLM Connectivity (Backend as a Proxy)
  provider        VARCHAR(50) NOT NULL,    -- 'ollama', 'openai', 'anthropic', 'vllm', etc.
  base_url        VARCHAR(255),            -- URL của Provider (vd: http://localhost:11434 hoặc https://api.openai.com)
  model           VARCHAR(100) NOT NULL,   -- Tên model (vd: 'llama3:8b', 'gpt-4o')
  
  -- Prompt Engineering
  system_prompt   TEXT NOT NULL,           -- "Hướng dẫn hành vi" cho Agent
  
  -- Flexible Configuration
  -- Lưu temperature, max_tokens, stop_sequences, hoặc các params đặc thù của Local LLM
  ai_settings     JSON,                    
  
  status          ENUM('active', 'disabled') DEFAULT 'active',
  
  created_at      TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at      TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);
```

---

## 🔗 Related Table: agent_documents (RAG Knowledge)
Bảng trung gian để nạp tài liệu nội bộ cho Agent.

```sql
CREATE TABLE agent_documents (
  agent_id        VARCHAR(36) NOT NULL REFERENCES agents(id),
  document_id     VARCHAR(36) NOT NULL REFERENCES documents(id),
  PRIMARY KEY (agent_id, document_id)
);
```

---

## 🎯 Purpose
Bảng này đóng vai trò là một **Danh mục (Registry)** các AI Agents khả dụng cho hệ thống/dự án.

---

## 🔗 Associations (Sequelize)

```typescript
// models/agent.model.ts
Agent.hasMany(Task, { foreignKey: 'generated_by_agent', as: 'generatedTasks' });
Agent.hasMany(Chat, { foreignKey: 'agent_id', as: 'chats' });
```

---

*Last Updated: 2026-02-15*
