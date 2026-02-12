# AI Agents Module

**Version:** v1  
**Date:** 2026-02-11  
**Skills:** `backend-architect`

---

## 🎯 Overview

AI Agents module quản lý AI agents (PM, Dev, Reviewer) trong organization.

**Status:** 📝 To Be Implemented  
**Pattern:** Follow [04-projects-module.md](04-projects-module.md)

---

## 📋 Implementation Guide

### Key DTOs

```typescript
export const CreateAgentSchema = z.object({
  name: z.string().min(1).max(100),
  role: z.enum(['pm', 'developer', 'reviewer', 'analyst', 'custom']),
  agent_external_key: z.string().min(1), // Key to external AI service
  project_id: z.string().uuid().optional(),
});
```

### Agent-Specific Features

- **Agent Registry** - Register external AI capabilities
- **Routing** - Call external AI via `agent_external_key`
- **Audit Tracking** - Track which agent created which project/task
- **No Internal Prompts** - Prompts/Models are managed externally

---

## 📚 Related Documents

- [04-projects-module.md](04-projects-module.md) - Base pattern
- [09-chat-module.md](09-chat-module.md) - Agent execution

---

*Status: Placeholder - Implement using Projects module pattern*
