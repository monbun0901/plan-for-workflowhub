# workflow_templates Table

**Type:** Automation Definition  
**Tenant Isolation:** ✅ Required (`organization_id`)

---

## 📋 Schema

```sql
CREATE TABLE workflow_templates (
  id              VARCHAR(36) PRIMARY KEY,
  organization_id VARCHAR(36) NOT NULL REFERENCES organizations(id),
  
  name            VARCHAR(100) NOT NULL,
  description     TEXT,
  
  -- Trigger Definition (Sự kiện kích hoạt)
  -- e.g., { "event": "issue.updated", "conditions": { "status_to": "done" } }
  trigger_config  JSON NOT NULL,
  
  -- Steps Definition (Các bước thực hiện)
  -- Cấu trúc mỗi bước:
  -- { 
  --   "step_name": "Review Code",
  --   "agent_id": "...", 
  --   "model_override": "gpt-4", 
  --   "tools_override": [...],
  --   "predefined_prompt": "...",
  --   "usage_tips": "...",
  --   "tags": ["review", "security"]
  -- }
  steps_config    JSON NOT NULL,
  
  status          ENUM('active', 'draft', 'archived') DEFAULT 'draft',
  
  created_by      VARCHAR(36) REFERENCES users(id),
  created_at      TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at      TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  
  INDEX idx_org_status (organization_id, status)
);
```

---

## 🎯 Purpose
Lưu trữ các kịch bản tự động hóa (Automations). Mỗi doanh nghiệp có thể tự định nghĩa các quy trình riêng để AI giúp họ làm việc hiệu quả hơn.

**Ví dụ:**
- **Auto-Review:** Khi Developer đẩy Code (Trigger) -> Gọi Agent Reviewer (Action) -> Post kết quả vào Chat (Action).
- **Auto-Summary:** Khi Project hoàn thành -> Gọi Agent Analyst để tạo báo cáo tổng kết.

---

*Last Updated: 2026-02-11*
