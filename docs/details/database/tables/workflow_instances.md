# workflow_instances Table

**Type:** Automation Run History  
**Tenant Isolation:** ✅ Required (`organization_id`)

---

## 📋 Schema

```sql
CREATE TABLE workflow_instances (
  id              VARCHAR(36) PRIMARY KEY,
  template_id     VARCHAR(36) NOT NULL REFERENCES workflow_templates(id),
  organization_id VARCHAR(36) NOT NULL REFERENCES organizations(id),
  
  -- Thông tin về entity kích hoạt workflow này
  -- e.g., "issue", "task", "document"
  trigger_entity_type VARCHAR(50) NOT NULL,
  trigger_entity_id   VARCHAR(36) NOT NULL,
  
  -- Trạng thái thực thi
  status          ENUM('running', 'completed', 'failed', 'partial_success') DEFAULT 'running',
  
  -- Kết quả chi tiết của từng bước
  -- e.g., [ { "step_1": "success", "output": "..." }, { "step_2": "failed", "error": "..." } ]
  execution_log   JSON,
  
  started_at      TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  completed_at    TIMESTAMP,
  
  INDEX idx_template (template_id),
  INDEX idx_org (organization_id)
);
```

---

## 🎯 Purpose
Theo dõi lịch sử chạy của các Automations. Giúp Admin biết được Workflow nào đang chạy, Workflow nào bị lỗi và do đâu.

---

*Last Updated: 2026-02-11*
