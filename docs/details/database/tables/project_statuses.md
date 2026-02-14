# project_statuses Table

**Type:** Lookup Table (Metadata)  
**Tenant Isolation:** N/A (Single-Tenant)

---

## 📋 Schema

```sql
CREATE TABLE project_statuses (
  id              VARCHAR(36) PRIMARY KEY,
  
  name            VARCHAR(50) NOT NULL, -- Ví dụ: "Active", "On Hold", "Completed", "Archived"
  description     TEXT,
  order_index     INT DEFAULT 0,
  
  state_type      ENUM('active', 'on_hold', 'completed', 'archived') NOT NULL,
  
  created_at      TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at      TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  
  INDEX idx_order (order_index)
);
```

---

## 🔗 Associations (Sequelize)

```typescript
// models/project-status.model.ts
ProjectStatus.hasMany(Project, {
  foreignKey: 'status_id',
  as: 'projects'
});
```

---

*Last Updated: 2026-02-15*
