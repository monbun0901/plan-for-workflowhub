# Tasks Module

**Version:** v1  
**Date:** 2026-02-11  
**Skills:** `backend-architect`, `nodejs-best-practices`

---

## 🎯 Overview

Tasks module quản lý tasks trong issues hoặc standalone.

**Status:** 📝 To Be Implemented  
**Pattern:** Follow [04-projects-module.md](04-projects-module.md) as reference

---

## 📋 Implementation Guide

Follow same pattern as Projects module with task-specific fields.

### Key DTOs

```typescript
export const CreateTaskSchema = z.object({
  title: z.string().min(1).max(255),
  description: z.string().optional(),
  status_id: z.string().uuid(), // Reference to workflow_statuses table
  issue_id: z.string().uuid().optional(),
  assignee_ids: z.array(z.string().uuid()).optional(), // Multi-assignee support
  due_date: z.string().datetime().optional(),
  priority: z.enum(['low', 'medium', 'high', 'critical']).default('medium'),
  estimated_hours: z.number().optional(),
});
```

**Note:** 
- Use `workflow_statuses` table (filtered by `target_type='task'`)
- Support multi-assignee via `task_assignees` junction table
- Track assignment history in `task_assignments` audit log

### Task-Specific Features

- **Kanban board** - Status transitions using workflow_statuses
- **Multi-assignee** - Support multiple users per task
- **Assignment history** - Audit trail of all assignments
- **Time tracking** - Estimated vs actual hours
- **Dependencies** - Tasks can depend on other tasks (future)

---

## 📚 Related Documents

- [04-projects-module.md](04-projects-module.md) - **Reference pattern**
- [05-issues-module.md](05-issues-module.md) - Parent issues
- [../database/tables/tasks.md](../../database/tables/tasks.md) - Database schema
- [../database/tables/workflow_statuses.md](../../database/tables/workflow_statuses.md) - Status table

---

*Status: Placeholder - Implement using Projects module pattern*
