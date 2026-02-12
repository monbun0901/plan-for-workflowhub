# Multi-Tenant Isolation

**Version:** v1  
**Date:** 2026-02-11  
**Skills:** `backend-architect`, `backend-security-coder`

---

## 🎯 Overview

WorkflowHub là multi-tenant application. **Every organization's data MUST be isolated** để prevent data leaks.

**Core Principle:** `organization_id` field trên mọi table và MUST be filtered trong mọi query.

---

## 🏢 Tenant Isolation Strategy

### Database-Level Isolation

```
┌─────────────────────────────────────────────────┐
│           ROW-LEVEL SECURITY (RLS)              │
├─────────────────────────────────────────────────┤
│                                                 │
│  Organization A                                 │
│  ├─ Project 1 (org_id: A)                      │
│  ├─ Project 2 (org_id: A)                      │
│  └─ Issue 1 (org_id: A)                        │
│                                                 │
│  Organization B                                 │
│  ├─ Project 3 (org_id: B)   ← ISOLATED         │
│  └─ Project 4 (org_id: B)                      │
│                                                 │
└─────────────────────────────────────────────────┘
```

**Rule:** Organization A CANNOT access Organization B's data, even with valid project ID.

---

## 🔒 Implementation Pattern

### Repository Pattern (MANDATORY)

```typescript
// ❌ BAD: No tenant filtering
async findById(id: string) {
  return Project.findByPk(id); // DANGEROUS! Can access any org's project
}

// ✅ GOOD: Always filter by organization_id
async findById(id: string, organizationId: string) {
  return Project.findOne({
    where: { 
      id, 
      organization_id: organizationId // CRITICAL: Tenant isolation
    },
  });
}
```

### Service Layer

```typescript
// apps/api/src/modules/projects/services/project.service.ts
export class ProjectService {
  constructor(private readonly projectRepo: ProjectRepository) {}

  /**
   * Get project by ID
   * ALWAYS requires organizationId for tenant isolation
   */
  async getProject(organizationId: string, projectId: string) {
    const project = await this.projectRepo.findById(projectId, organizationId);
    
    if (!project) {
      throw new AppError(404, 'Project not found');
    }
    
    return project;
  }
}
```

### Controller Layer

```typescript
// apps/api/src/modules/projects/controllers/project.controller.ts
export class ProjectController {
  /**
   * GET /api/organizations/:orgId/projects/:id
   * Organization ID comes from URL, validated by middleware
   */
  async getProject(req: Request, res: Response, next: NextFunction) {
    try {
      const { orgId, id } = req.params;
      
      // Verify user has access to this organization
      if (!req.user.organizationIds.includes(orgId)) {
        return res.status(403).json({
          success: false,
          error: { code: 'FORBIDDEN', message: 'Access denied' },
        });
      }
      
      const project = await this.projectService.getProject(orgId, id);
      
      res.json({ success: true, data: project });
    } catch (error) {
      next(error);
    }
  }
}
```

---

## 🛡️ Tenant Middleware

### Organization Access Check

```typescript
// apps/api/src/shared/middleware/tenant.middleware.ts
import { Request, Response, NextFunction } from 'express';

/**
 * Tenant middleware - Verify user has access to organization
 * Apply this BEFORE any organization-scoped routes
 */
export async function verifyOrganizationAccess(
  req: Request,
  res: Response,
  next: NextFunction
) {
  try {
    const { orgId } = req.params;
    const userId = req.user.id;
    
    // Check if user is member of this organization
    const member = await Member.findOne({
      where: {
        organization_id: orgId,
        user_id: userId,
        status: 'active',
      },
      include: [{ 
        model: Role, 
        as: 'role',
        attributes: ['name', 'permissions']
      }]
    });
    
    if (!member) {
      return res.status(403).json({
        success: false,
        error: {
          code: 'ORGANIZATION_ACCESS_DENIED',
          message: 'You do not have access to this organization',
        },
      });
    }
    
    // Attach organization context to request
    req.organization = {
      id: orgId,
      role: member.role.name,
      permissions: member.role.permissions, // Dynamic RBAC
    };
    
    next();
  } catch (error) {
    next(error);
  }
}
```

**Usage:**
```typescript
// All organization routes MUST use this middleware
router.use('/api/organizations/:orgId', 
  authenticate, 
  verifyOrganizationAccess
);

router.get('/api/organizations/:orgId/projects', projectController.list);
```

---

## 📋 Tenant Isolation Checklist

### For Every Feature Module

- [ ] **All tables** have `organization_id` column
- [ ] **All queries** filter by `organization_id`
- [ ] **Repository methods** require `organizationId` parameter
- [ ] **Service methods** accept and pass `organizationId`
- [ ] **Controllers** extract `orgId` from URL params
- [ ] **Middleware** validates user access to organization
- [ ] **Indexes** include `organization_id` for performance

### Database Schema

```sql
-- Example: projects table
CREATE TABLE projects (
  id UUID PRIMARY KEY,
  organization_id UUID NOT NULL,  -- REQUIRED
  name VARCHAR(255) NOT NULL,
  -- ... other fields
  
  -- CRITICAL: Index for tenant queries
  INDEX idx_projects_org_id (organization_id),
  
  -- Composite index for common queries
  INDEX idx_projects_org_status (organization_id, status),
  
  FOREIGN KEY (organization_id) REFERENCES organizations(id) ON DELETE CASCADE
);
```

---

## ⚠️ Common Pitfalls

### ❌ Pitfall 1: Forgetting organization_id Filter

```typescript
// ❌ DANGEROUS: Can return any organization's data
async searchProjects(query: string) {
  return Project.findAll({
    where: { name: { [Op.like]: `%${query}%` } }
  });
}

// ✅ CORRECT: Always filter by organization
async searchProjects(organizationId: string, query: string) {
  return Project.findAll({
    where: { 
      organization_id: organizationId,  // CRITICAL
      name: { [Op.like]: `%${query}%` }
    }
  });
}
```

### ❌ Pitfall 2: Direct ID Access in URL

```typescript
// ❌ BAD: /api/projects/:id (no org context)
router.get('/api/projects/:id', projectController.get);

// ✅ GOOD: /api/organizations/:orgId/projects/:id
router.get('/api/organizations/:orgId/projects/:id', projectController.get);
```

### ❌ Pitfall 3: Join Queries Without Tenant Filter

```typescript
// ❌ DANGEROUS: Join without tenant filter
async getProjectWithIssues(projectId: string) {
  return Project.findByPk(projectId, {
    include: [{ model: Issue }]  // Issues from ANY org!
  });
}

// ✅ CORRECT: Filter both project and issues
async getProjectWithIssues(organizationId: string, projectId: string) {
  return Project.findOne({
    where: { id: projectId, organization_id: organizationId },
    include: [{
      model: Issue,
      where: { organization_id: organizationId }  // CRITICAL
    }]
  });
}
```

---

## 🧪 Testing Tenant Isolation

### Integration Tests

```typescript
// apps/api/src/modules/projects/__tests__/tenant-isolation.test.ts
describe('Tenant Isolation - Projects', () => {
  let orgA: Organization;
  let orgB: Organization;
  let projectA: Project;
  let projectB: Project;
  
  beforeEach(async () => {
    orgA = await Organization.create({ name: 'Org A' });
    orgB = await Organization.create({ name: 'Org B' });
    
    projectA = await Project.create({
      name: 'Project A',
      organization_id: orgA.id,
    });
    
    projectB = await Project.create({
      name: 'Project B',
      organization_id: orgB.id,
    });
  });
  
  it('should NOT allow orgA user to access orgB project', async () => {
    const projectRepo = new ProjectRepository();
    
    // Try to access orgB's project with orgA's context
    const result = await projectRepo.findById(projectB.id, orgA.id);
    
    expect(result).toBeNull(); // Should not find it
  });
  
  it('should allow orgB user to access orgB project', async () => {
    const projectRepo = new ProjectRepository();
    
    const result = await projectRepo.findById(projectB.id, orgB.id);
    
    expect(result).not.toBeNull();
    expect(result.id).toBe(projectB.id);
  });
});
```

---

## 🎯 Best Practices

### 1. URL Structure

```
✅ GOOD: /api/organizations/:orgId/projects
✅ GOOD: /api/organizations/:orgId/projects/:projectId
✅ GOOD: /api/organizations/:orgId/issues

❌ BAD: /api/projects (no org context)
❌ BAD: /api/projects/:id (easy to leak data)
```

### 2. Database Queries

```typescript
// Always use organizationId in WHERE clause
where: { 
  organization_id: organizationId,  // FIRST
  id: projectId,                     // THEN specific filter
}
```

### 3. Sequelize Scopes (Optional but Recommended)

```typescript
// apps/api/src/modules/projects/models/project.model.ts
Project.addScope('forOrganization', (organizationId: string) => ({
  where: { organization_id: organizationId }
}));

// Usage:
const projects = await Project.scope({ 
  method: ['forOrganization', orgId] 
}).findAll();
```

---

## 🔧 Migration Guide

### Adding organization_id to Existing Table

```sql
-- Step 1: Add column (nullable first)
ALTER TABLE projects 
ADD COLUMN organization_id UUID NULL;

-- Step 2: Set default org for existing data (if needed)
UPDATE projects 
SET organization_id = (SELECT id FROM organizations LIMIT 1);

-- Step 3: Make it NOT NULL
ALTER TABLE projects 
MODIFY COLUMN organization_id UUID NOT NULL;

-- Step 4: Add foreign key
ALTER TABLE projects
ADD CONSTRAINT fk_projects_organization
FOREIGN KEY (organization_id) REFERENCES organizations(id) ON DELETE CASCADE;

-- Step 5: Add index
CREATE INDEX idx_projects_org_id ON projects(organization_id);
```

---

## 📚 Related Documents

- [01-architecture.md](01-architecture.md) - Backend architecture
- [02-authentication.md](02-authentication.md) - User authentication
- [04-projects-module.md](04-projects-module.md) - Example implementation
- [../../basics/step-5-data-design.md](../../basics/step-5-data-design.md) - Database schema

---

## 🎯 Skill Usage

```
@backend-architect @backend-security-coder
Implement multi-tenant isolation:
- organization_id on all tables
- Row-level filtering in repositories
- Tenant middleware validation
- Integration tests for isolation
```

---

*Document Version: 1.0*  
*Last Updated: 2026-02-11*
