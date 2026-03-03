# workflow_templates Table

**Type:** Automation Blueprint  
**Tenant Isolation:** N/A (Single-Tenant)

---

## 📋 Schema

```sql
CREATE TABLE workflow_templates (
  id              VARCHAR(36) PRIMARY KEY,
  
  name            VARCHAR(100) NOT NULL,
  description     VARCHAR(1000),
  
  -- ── Trigger: Khi nào workflow được kích hoạt? ──
  -- JSON: { "type": "manual|scheduled|event", "event": "issue_created", "schedule": "0 9 * * 1" }
  trigger_config  JSON NOT NULL,
  
  -- ── Input Schema: Workflow cần input gì? ──
  -- JSON: { "fields": [{ "key": "transcript", "type": "textarea", "label": "Meeting Notes", "required": true }] }
  input_schema    JSON,
  
  -- ── Steps: Chuỗi bước thực thi ──
  -- JSON Array: Mỗi bước gán Agent + Model + Prompt riêng
  -- [{ 
  --   "step_order": 1, 
  --   "name": "Analyze",
  --   "agent_id": "uuid",
  --   "model_override": { "provider": "openai", "model": "gpt-4o" },
  --   "prompt_template": "Phân tích: {{input}}",
  --   "input_mapping": { "type": "workflow_input", "source": "transcript" },
  --   "conditions": [],
  --   "timeout_ms": 60000,
  --   "retry": { "max_attempts": 2, "delay_ms": 5000 }
  -- }]
  steps_config    JSON NOT NULL,
  
  -- ── Meta ──
  status          ENUM('active', 'draft', 'archived') DEFAULT 'draft',
  created_by      VARCHAR(36) REFERENCES users(id),
  
  -- ── Usage Stats ──
  total_runs      INT DEFAULT 0,
  last_run_at     TIMESTAMP NULL,
  avg_duration_ms INT DEFAULT 0,
  
  created_at      TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at      TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  
  INDEX idx_status (status),
  INDEX idx_created_by (created_by)
);
```

---

## 🔗 Associations (Sequelize)

```typescript
// models/workflow-template.model.ts

// Categories
WorkflowTemplate.belongsToMany(WorkflowCategory, {
  through: 'workflow_category_mappings',
  foreignKey: 'workflow_id',
  otherKey: 'category_id',
  as: 'categories'
});

// Instances (execution history)
WorkflowTemplate.hasMany(WorkflowInstance, {
  foreignKey: 'template_id',
  as: 'instances'
});

// Creator
WorkflowTemplate.belongsTo(User, { foreignKey: 'created_by', as: 'creator' });
```

---

## 🎯 steps_config Detail

Mỗi phần tử trong `steps_config` JSON array:

| Field | Type | Required | Description |
|---|---|---|---|
| `step_order` | number | ✅ | Thứ tự thực thi (1-based) |
| `name` | string | ✅ | Tên bước |
| `agent_id` | uuid | ✅ | Agent thực thi bước này |
| `model_override` | object | ❌ | Ghi đè model mặc định của Agent |
| `model_override.provider` | string | ❌ | Provider: openai, anthropic, google, ollama |
| `model_override.model` | string | ❌ | Model name: gpt-4o, claude-3.5-sonnet |
| `prompt_template` | string | ✅ | Template prompt (hỗ trợ `{{variables}}`) |
| `input_mapping` | object | ✅ | Nguồn input cho bước này |
| `conditions` | array | ❌ | Điều kiện chạy bước |
| `timeout_ms` | number | ❌ | Timeout (default: 60000ms) |
| `retry` | object | ❌ | Retry policy |

---

*Last Updated: 2026-02-25*
