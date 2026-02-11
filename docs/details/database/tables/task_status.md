# task_status Table

**Type:** Reference / Configuration Table  
**Tenant Isolation:** ✅ Required (`organization_id`)

---

## 📋 Schema

```sql
CREATE TABLE task_status (
  id              VARCHAR(36) PRIMARY KEY,
  organization_id VARCHAR(36) NOT NULL REFERENCES organizations(id) ON DELETE CASCADE,
  
  name            VARCHAR(100) NOT NULL,         -- "To Do", "In Progress", "Done"
  slug            VARCHAR(100) NOT NULL,         -- "todo", "in_progress", "done"
  color           VARCHAR(7) NOT NULL,           -- Badge color: #3B82F6
  icon            VARCHAR(50),                   -- Optional icon name
  order           INT NOT NULL DEFAULT 0,        -- Display order
  is_default      BOOLEAN DEFAULT FALSE,         -- Default status on task creation
  is_final        BOOLEAN DEFAULT FALSE,         -- Is this a completion status?
  
  created_at      TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at      TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  
  UNIQUE KEY unique_org_slug (organization_id, slug),
  INDEX idx_org_order (organization_id, `order`)
);
```

---

## 🎯 Purpose

**Configurable status workflow per organization:**
- **Custom Statuses:** Each org can define own workflow (e.g. "Backlog", "Ready", "In Progress", "Review", "Done").
- **Visual Identity:** Color & icon per status for UI badges.
- **Order Control:** Display order in dropdowns and kanban boards.
- **Default Status:** Auto-assign when task is created.
- **Final State Detection:** Mark statuses that represent completion (for metrics).

---

## 📊 Standard Seed Data (Default Statuses)

```javascript
const DEFAULT_STATUSES = [
  { name: 'To Do',        slug: 'todo',         color: '#6B7280', order: 1, is_default: true },
  { name: 'In Progress',  slug: 'in_progress',  color: '#3B82F6', order: 2 },
  { name: 'In Review',    slug: 'in_review',    color: '#F59E0B', order: 3 },
  { name: 'Done',         slug: 'done',         color: '#10B981', order: 4, is_final: true },
];
```

---

## 🔗 Relationships

### tasks.status_id → task_status.id
```sql
ALTER TABLE tasks 
ADD COLUMN status_id VARCHAR(36) REFERENCES task_status(id);

-- Migration: map old string status to new IDs
UPDATE tasks t
SET t.status_id = (
  SELECT id FROM task_status ts 
  WHERE ts.slug = t.status AND ts.organization_id = t.organization_id 
  LIMIT 1
);
```

---

## 🛡️ Security

- **Tenant Isolation:** Status luôn thuộc organization cụ thể.
- **Prevent Deletion:** Không cho xóa status đang được dùng (check constraint hoặc soft delete).

---

## 📡 API Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/task-statuses` | List all statuses for org |
| `POST` | `/task-statuses` | Create custom status (admin only) |
| `PATCH` | `/task-statuses/:id` | Update status (admin only) |
| `DELETE` | `/task-statuses/:id` | Delete if not in use |

---

## 💡 Example Query: Task Count by Status

```sql
SELECT 
  ts.name,
  ts.color,
  COUNT(t.id) as task_count
FROM task_status ts
LEFT JOIN tasks t ON t.status_id = ts.id AND t.project_id = :projectId
WHERE ts.organization_id = :orgId
GROUP BY ts.id
ORDER BY ts.order;
```

---

*Last Updated: 2026-02-11*
