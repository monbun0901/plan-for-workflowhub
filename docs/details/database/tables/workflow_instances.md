# workflow_instances Table

**Type:** Automation Execution History (Full Trace)  
**Tenant Isolation:** N/A (Single-Tenant)

---

## 📋 Schema

```sql
CREATE TABLE workflow_instances (
  id              VARCHAR(36) PRIMARY KEY,
  template_id     VARCHAR(36) NOT NULL REFERENCES workflow_templates(id),
  
  -- ── Trigger Context ──
  triggered_by    VARCHAR(36) NOT NULL REFERENCES users(id),
  trigger_entity_type VARCHAR(50),             -- 'manual', 'scheduled', 'issue', 'document'
  trigger_entity_id   VARCHAR(36),             -- ID entity kích hoạt (nếu event-based)
  
  -- ── Input Data ──
  -- JSON: Dữ liệu đầu vào cho lần chạy này (theo input_schema của template)
  input_data      JSON,
  
  -- ── Execution Status ──
  status          ENUM('pending', 'running', 'completed', 'failed', 'partial_success', 'cancelled') DEFAULT 'pending',
  current_step    INT DEFAULT 0,               -- Step đang chạy (0 = chưa bắt đầu)
  
  -- ── Step Results (Full Traceability) ──
  -- JSON Array: Kết quả chi tiết từng bước
  -- [{
  --   "step_order": 1,
  --   "name": "Analyze Requirements",
  --   "agent_id": "uuid",
  --   "agent_name": "Analyst Agent",
  --   "model_used": "gpt-4o",
  --   "status": "completed",
  --   "input_preview": "First 200 chars...",
  --   "output": "Full output text...",
  --   "tokens_used": 1523,
  --   "duration_ms": 8432,
  --   "error": null,
  --   "started_at": "2026-02-25T10:00:00Z",
  --   "completed_at": "2026-02-25T10:00:08Z"
  -- }]
  step_results    JSON,
  
  -- ── Aggregate Stats ──
  total_tokens_used   INT DEFAULT 0,
  total_duration_ms   INT DEFAULT 0,
  
  -- ── Error Info ──
  error_message   TEXT,
  
  -- ── Link tới Chat (theo dõi/thảo luận) ──
  chat_id         VARCHAR(36) REFERENCES chats(id),
  
  -- ── Scope ──
  project_id      VARCHAR(36) REFERENCES projects(id),
  
  -- ── Timestamps ──
  started_at      TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  completed_at    TIMESTAMP,
  
  INDEX idx_template (template_id),
  INDEX idx_triggered_by (triggered_by),
  INDEX idx_status (status),
  INDEX idx_project (project_id)
);
```

---

## 🔗 Associations (Sequelize)

```typescript
// models/workflow-instance.model.ts

// Template
WorkflowInstance.belongsTo(WorkflowTemplate, { foreignKey: 'template_id', as: 'template' });

// User who triggered
WorkflowInstance.belongsTo(User, { foreignKey: 'triggered_by', as: 'triggeredBy' });

// Chat (discussion thread)
WorkflowInstance.belongsTo(Chat, { foreignKey: 'chat_id', as: 'chat' });

// Project scope
WorkflowInstance.belongsTo(Project, { foreignKey: 'project_id', as: 'project' });
```

---

## 📊 step_results JSON Detail

| Field | Type | Description |
|---|---|---|
| `step_order` | number | Thứ tự bước |
| `name` | string | Tên bước |
| `agent_id` | string | UUID Agent thực thi |
| `agent_name` | string | Tên Agent (snapshot at execution time) |
| `model_used` | string | Model thực tế đã dùng (có thể khác default nếu override) |
| `status` | string | `completed`, `failed`, `skipped` |
| `input_preview` | string | 200 ký tự đầu của input |
| `output` | string | Full output text |
| `tokens_used` | number | Tokens consumed |
| `duration_ms` | number | Execution time |
| `error` | string? | Error message nếu failed |
| `started_at` | string | ISO timestamp |
| `completed_at` | string | ISO timestamp |

→ Cho phép **truy vết, so sánh, tối ưu** từng bước qua nhiều lần chạy.

---

*Last Updated: 2026-02-25*
