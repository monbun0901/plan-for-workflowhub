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
  status: z.enum(['todo', 'in_progress', 'done']).default('todo'),
  issue_id: z.string().uuid().optional(),
  assignee_id: z.string().uuid().optional(),
  due_date: z.string().datetime().optional(),
});
```

### Task-Specific Features

- **Kanban board** - Status transitions (todo → in_progress → done)
- **Dependencies** - Tasks can depend on other tasks
- **Time tracking** - Estimated vs actual hours
- **Checklist items** - Sub-tasks within a task

---

## 📚 Related Documents

- [04-projects-module.md](04-projects-module.md) - **Reference pattern**
- [05-issues-module.md](05-issues-module.md) - Parent issues

---

*Status: Placeholder - Implement using Projects module pattern*
