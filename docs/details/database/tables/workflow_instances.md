# workflow_instances Table

**Type:** Automation Run History  
**Tenant Isolation:** N/A (Single-Tenant)

---

## 📋 Schema

```sql
CREATE TABLE workflow_instances (
  id              VARCHAR(36) PRIMARY KEY,
  template_id     VARCHAR(36) NOT NULL REFERENCES workflow_templates(id),
  
  -- Thông tin về entity kích hoạt workflow này
  trigger_entity_type VARCHAR(50) NOT NULL,
  trigger_entity_id   VARCHAR(36) NOT NULL,
  
  -- Trạng thái thực thi
  status          ENUM('running', 'completed', 'failed', 'partial_success') DEFAULT 'running',
  
  -- Kết quả chi tiết của từng bước
  execution_log   JSON,
  
  -- Link tới phiên chat để thảo luận/theo dõi tiến độ automation
  chat_id         VARCHAR(36) REFERENCES chats(id),
  
  started_at      TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  completed_at    TIMESTAMP,
  
  INDEX idx_template (template_id)
);
```

---

*Last Updated: 2026-02-15*
