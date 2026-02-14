# workflow_templates Table

**Type:** Automation Definition  
**Tenant Isolation:** N/A (Single-Tenant)

---

## 📋 Schema

```sql
CREATE TABLE workflow_templates (
  id              VARCHAR(36) PRIMARY KEY,
  
  name            VARCHAR(100) NOT NULL,
  description     TEXT,
  
  -- Trigger Definition
  trigger_config  JSON NOT NULL,
  
  -- Steps Definition
  -- Mỗi bước định nghĩa 1 Agent chuyên biệt và prompt riêng.
  -- JSON: [{ "step": 1, "agent_id": "...", "prompt": "..." }, ...]
  steps_config    JSON NOT NULL,
  
  created_by      VARCHAR(36) REFERENCES users(id),
  created_at      TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at      TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);
```

---

## 🔗 Associations (Sequelize)

```typescript
// models/workflow-template.model.ts
WorkflowTemplate.belongsToMany(WorkflowCategory, {
  through: 'workflow_category_mappings',
  foreignKey: 'workflow_id',
  otherKey: 'category_id',
  as: 'categories'
});
```

---

*Last Updated: 2026-02-15*
