# Issues Module

**Version:** v1  
**Date:** 2026-02-11  
**Skills:** `backend-architect`, `nodejs-best-practices`

---

## 🎯 Overview

Issues module quản lý issues/bugs trong projects.

**Status:** 📝 To Be Implemented  
**Pattern:** Follow [04-projects-module.md](04-projects-module.md) as reference

---

## 📋 Implementation Guide

### Step 1: Copy Projects Module Structure

```bash
cp -r src/modules/projects src/modules/issues
```

### Step 2: Replace "Project" with "Issue"

- ProjectRepository → IssueRepository
- ProjectService → IssueService
- ProjectController → IssueController

### Step 3: Update DTOs

```typescript
// apps/api/src/modules/issues/dtos/create-issue.dto.ts
import { z } from 'zod';

export const CreateIssueSchema = z.object({
  title: z.string().min(1).max(255),
  description: z.string().optional(),
  status: z.enum(['open', 'in_progress', 'resolved', 'closed']).default('open'),
  priority: z.enum(['low', 'medium', 'high', 'critical']).default('medium'),
  assignee_id: z.string().uuid().optional(),
  project_id: z.string().uuid(),
});

export type CreateIssueDto = z.infer<typeof CreateIssueSchema>;
```

### Step 4: Issue-Specific Business Logic

```typescript
// apps/api/src/modules/issues/services/issue.service.ts
export class IssueService {
  // Same pattern as ProjectService
  
  /**
   * Assign issue to user
   */
  async assignIssue(
    organizationId: string, 
    issueId: string, 
    assigneeId: string
  ) {
    const issue = await this.issueRepo.findById(issueId, organizationId);
    
    if (!issue) {
      throw new AppError(404, 'Issue not found');
    }
    
    // Verify assignee is in same organization
    const member = await OrganizationMember.findOne({
      where: { user_id: assigneeId, organization_id: organizationId }
    });
    
    if (!member) {
      throw new AppError(400, 'Assignee not in organization');
    }
    
    return this.issueRepo.update(issueId, organizationId, { 
      assignee_id: assigneeId 
    });
  }
}
```

### Step 5: Register Routes

```typescript
// apps/api/src/app.ts
import { issueRoutes } from './modules/issues';

app.use(issueRoutes);
```

---

## 🗄️ Database Schema

See [../../basics/step-5-data-design.md](../../basics/step-5-data-design.md) for full schema.

**Key fields:**
- `id` - UUID primary key
- `organization_id` - Tenant isolation
- `project_id` - Parent project
- `title` - Issue title
- `description` - Detailed description
- `status` - open, in_progress, resolved, closed
- `priority` - low, medium, high, critical
- `assignee_id` - Assigned user

---

## 📚 Related Documents

- [04-projects-module.md](04-projects-module.md) - **Reference pattern**
- [01-architecture.md](01-architecture.md) - Architecture
- [03-multi-tenant.md](03-multi-tenant.md) - Tenant isolation

---

*Document Version: 1.0*  
*Last Updated: 2026-02-11*  
*Status: Placeholder - Implement using Projects module pattern*
