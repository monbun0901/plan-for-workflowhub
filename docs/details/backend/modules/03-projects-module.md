# Projects Module (Reference Implementation)

**Version:** v1  
**Date:** 2026-02-11  
**Skills:** `backend-architect`, `nodejs-best-practices`

---

## 🎯 Overview

Projects module là **reference implementation** cho tất cả feature modules khác trong WorkflowHub.

**Pattern:** Controller → Service → Repository

**Use this as template** khi implement Issues, Tasks, Documents, etc.

---

## 📁 Module Structure

```
apps/api/src/modules/projects/
├── controllers/
│   └── project.controller.ts      # HTTP handling
├── services/
│   └── project.service.ts         # Business logic
├── repositories/
│   └── project.repository.ts      # Data access
├── models/
│   └── project.model.ts           # Sequelize model
├── dtos/
│   ├── create-project.dto.ts      # Input validation
│   └── update-project.dto.ts
├── routes/
│   └── project.routes.ts          # Route definitions
└── index.ts                        # Module exports
```

---

## 🗄️ Repository Layer

```typescript
// apps/api/src/modules/projects/repositories/project.repository.ts
import { Project } from '../models/project.model';

/**
 * ProjectRepository - Data access layer for projects
 * @follows backend-architect: Repository pattern
 */
export class Project Repository {
  /**
   * Find project by ID with tenant check
   * @param id - Project ID
   * @param organizationId - Tenant ID
   * @returns Project or null
   */
  async findById(id: string, organizationId: string): Promise<Project | null> {
    return Project.findOne({
      where: { 
        id, 
        organization_id: organizationId // CRITICAL: Tenant isolation
      },
    });
  }

  /**
   * List projects for organization with pagination
   * @param organizationId - Tenant ID
   * @param page - Page number
   * @param limit - Items per page
   * @returns Projects with pagination meta
   */
  async list(organizationId: string, page: number = 1, limit: number = 20) {
    const offset = (page - 1) * limit;
    
    const { count, rows } = await Project.findAndCountAll({
      where: { organization_id: organizationId },
      limit,
      offset,
      order: [['created_at', 'DESC']],
    });
    
    return {
      data: rows,
      meta: {
        page,
        limit,
        total: count,
        totalPages: Math.ceil(count / limit),
      },
    };
  }

  /**
   * Create project
   * @param data - Project data
   * @returns Created project
   */
  async create(data: Partial<Project>): Promise<Project> {
    return Project.create(data);
  }

  /**
   * Update project with tenant check
   * @param id - Project ID
   * @param organizationId - Tenant ID
   * @param data - Update data
   * @returns Updated project or null
   */
  async update(
    id: string, 
    organizationId: string, 
    data: Partial<Project>
  ): Promise<Project | null> {
    const project = await this.findById(id, organizationId);
    if (!project) return null;
    
    await project.update(data);
    return project;
  }

  /**
   * Delete project with tenant check
   * @param id - Project ID
   * @param organizationId - Tenant ID
   * @returns True if deleted
   */
  async delete(id: string, organizationId: string): Promise<boolean> {
    const project = await this.findById(id, organizationId);
    if (!project) return false;
    
    await project.destroy();
    return true;
  }
}
```

**Key Points:**
- ✅ Every method requires `organizationId`
- ✅ Tenant filtering in WHERE clause
- ✅ Returns null if not found (service handles error)

---

## ⚙️ Service Layer

```typescript
// apps/api/src/modules/projects/services/project.service.ts
import { ProjectRepository } from '../repositories/project.repository';
import { CreateProjectDto, UpdateProjectDto } from '../dtos';

/**
 * ProjectService - Business logic layer
 * @follows backend-architect: Service pattern
 */
export class ProjectService {
  constructor(private readonly projectRepo: ProjectRepository) {}

  /**
   * List projects for organization
   */
  async listProjects(organizationId: string, page: number, limit: number) {
    return this.projectRepo.list(organizationId, page, limit);
  }

  /**
   * Get project by ID
   */
  async getProject(organizationId: string, projectId: string) {
    const project = await this.projectRepo.findById(projectId, organizationId);
    
    if (!project) {
      throw new AppError(404, 'Project not found');
    }
    
    return project;
  }

  /**
   * Create new project
   */
  async createProject(organizationId: string, dto: CreateProjectDto) {
    // Business logic: Generate slug from name
    const slug = dto.name.toLowerCase().replace(/\s+/g, '-');
    
    return this.projectRepo.create({
      ...dto,
      slug,
      organization_id: organizationId,
    });
  }

  /**
   * Update project
   */
  async updateProject(
    organizationId: string,
    projectId: string,
    dto: UpdateProjectDto
  ) {
    const project = await this.projectRepo.update(projectId, organizationId, dto);
    
    if (!project) {
      throw new AppError(404, 'Project not found');
    }
    
    return project;
  }

  /**
   * Delete project
   */
  async deleteProject(organizationId: string, projectId: string) {
    const deleted = await this.projectRepo.delete(projectId, organizationId);
    
    if (!deleted) {
      throw new AppError(404, 'Project not found');
    }
    
    return { success: true };
  }
}
```

**Key Points:**
- ✅ Business logic here (e.g., slug generation)
- ✅ Error handling (throws AppError)
- ✅ No HTTP concerns
- ✅ Calls repository for data access

---

## 🎮 Controller Layer

```typescript
// apps/api/src/modules/projects/controllers/project.controller.ts
import { Request, Response, NextFunction } from 'express';
import { ProjectService } from '../services/project.service';
import { CreateProjectDto, UpdateProjectDto } from '../dtos';

/**
 * ProjectController - HTTP request handling
 * @follows nodejs-best-practices: Controller pattern
 */
export class ProjectController {
  constructor(private readonly projectService: ProjectService) {}

  /**
   * GET /api/organizations/:orgId/projects
   */
  async listProjects(req: Request, res: Response, next: NextFunction) {
    try {
      const { orgId } = req.params;
      const page = parseInt(req.query.page as string) || 1;
      const limit = parseInt(req.query.limit as string) || 20;
      
      const result = await this.projectService.listProjects(orgId, page, limit);
      
      res.json({
        success: true,
        data: result.data,
        meta: result.meta,
      });
    } catch (error) {
      next(error);
    }
  }

  /**
   * POST /api/organizations/:orgId/projects
   */
  async createProject(req: Request, res: Response, next: NextFunction) {
    try {
      const { orgId } = req.params;
      const dto: CreateProjectDto = req.body;
      
      const project = await this.projectService.createProject(orgId, dto);
      
      res.status(201).json({
        success: true,
        data: project,
      });
    } catch (error) {
      next(error);
    }
  }

  /**
   * PUT /api/organizations/:orgId/projects/:id
   */
  async updateProject(req: Request, res: Response, next: NextFunction) {
    try {
      const { orgId, id } = req.params;
      const dto: UpdateProjectDto = req.body;
      
      const project = await this.projectService.updateProject(orgId, id, dto);
      
      res.json({
        success: true,
        data: project,
      });
    } catch (error) {
      next(error);
    }
  }

  /**
   * DELETE /api/organizations/:orgId/projects/:id
   */
  async deleteProject(req: Request, res: Response, next: NextFunction) {
    try {
      const { orgId, id } = req.params;
      
      await this.projectService.deleteProject(orgId, id);
      
      res.status(204).send();
    } catch (error) {
      next(error);
    }
  }
}
```

**Key Points:**
- ✅ Thin controller - no business logic
- ✅ Extract params from req
- ✅ Call service layer
- ✅ Standard response format
- ✅ Error handling via next()

---

## 📝 DTOs (Data Transfer Objects)

### Create DTO

```typescript
// apps/api/src/modules/projects/dtos/create-project.dto.ts
import { z } from 'zod';

export const CreateProjectSchema = z.object({
  name: z.string().min(1).max(255),
  description: z.string().optional(),
  status: z.enum(['active', 'archived']).default('active'),
});

export type CreateProjectDto = z.infer<typeof CreateProjectSchema>;
```

### Update DTO

```typescript
// apps/api/src/modules/projects/dtos/update-project.dto.ts
import { z } from 'zod';

export const UpdateProjectSchema = z.object({
  name: z.string().min(1).max(255).optional(),
  description: z.string().optional(),
  status: z.enum(['active', 'archived']).optional(),
});

export type UpdateProjectDto = z.infer<typeof UpdateProjectSchema>;
```

---

## 🛣️ Routes

```typescript
// apps/api/src/modules/projects/routes/project.routes.ts
import { Router } from 'express';
import { validate } from '../../../shared/middleware/validate.middleware';
import { authenticate } from '../../../shared/middleware/auth.middleware';
import { verifyOrganizationAccess } from '../../../shared/middleware/tenant.middleware';
import { projectController } from '../index';
import { CreateProjectSchema, UpdateProjectSchema } from '../dtos';

const router = Router();

// All routes require authentication and org access
router.use(authenticate, verifyOrganizationAccess);

router.get(
  '/api/organizations/:orgId/projects',
  projectController.listProjects.bind(projectController)
);

router.post(
  '/api/organizations/:orgId/projects',
  validate(CreateProjectSchema),
  projectController.createProject.bind(projectController)
);

router.get(
  '/api/organizations/:orgId/projects/:id',
  projectController.getProject.bind(projectController)
);

router.put(
  '/api/organizations/:orgId/projects/:id',
  validate(UpdateProjectSchema),
  projectController.updateProject.bind(projectController)
);

router.delete(
  '/api/organizations/:orgId/projects/:id',
  projectController.deleteProject.bind(projectController)
);

export { router as projectRoutes };
```

---

## 🔧 Module Composition

```typescript
// apps/api/src/modules/projects/index.ts
import { ProjectRepository } from './repositories/project.repository';
import { ProjectService } from './services/project.service';
import { ProjectController } from './controllers/project.controller';

// Initialize dependencies
const projectRepo = new ProjectRepository();
const projectService = new ProjectService(projectRepo);
const projectController = new ProjectController(projectService);

export { projectController };
export { projectRoutes } from './routes/project.routes';
```

---

## 🧪 Testing

### Repository Tests

```typescript
// apps/api/src/modules/projects/repositories/__tests__/project.repository.test.ts
describe('ProjectRepository', () => {
  let repo: ProjectRepository;
  let orgId: string;
  
  beforeEach(() => {
    repo = new ProjectRepository();
    orgId = 'test-org-id';
  });
  
  it('should create project', async () => {
    const data = {
      name: 'Test Project',
      organization_id: orgId,
    };
    
    const project = await repo.create(data);
    
    expect(project.name).toBe('Test Project');
    expect(project.organization_id).toBe(orgId);
  });
  
  it('should filter by organizationId', async () => {
    // Create project for org A
    const projectA = await repo.create({
      name: 'Project A',
      organization_id: 'org-a',
    });
    
    // Try to find with org B context
    const result = await repo.findById(projectA.id, 'org-b');
    
    expect(result).toBeNull(); // Should not find
  });
});
```

---

## 🎯 Replication Guide

### To create Issues module (copy this pattern):

1. **Copy project structure** → `src/modules/issues/`
2. **Replace "Project" with "Issue"** in all files
3. **Update DTOs** with issue-specific fields
4. **Add issue-specific business logic** in service
5. **Register routes** in main app

### Example: Issues Module

```typescript
// apps/api/src/modules/issues/repositories/issue.repository.ts
export class IssueRepository {
  async findById(id: string, organizationId: string) {
    return Issue.findOne({
      where: { id, organization_id: organizationId }
    });
  }
  
  // ... same pattern as ProjectRepository
}
```

---

## 📚 Related Documents

- [01-architecture.md](01-architecture.md) - Layered architecture
- [02-authentication.md](02-authentication.md) - Auth middleware
- [03-multi-tenant.md](03-multi-tenant.md) - Tenant isolation
- [../../basics/step-5-data-design.md](../../basics/step-5-data-design.md) - Database schema

---

## 🎯 Skill Usage

```
@backend-architect @nodejs-best-practices
Create {feature} module following projects pattern:
- Repository with tenant isolation
- Service with business logic
- Controller with HTTP handling
- DTOs with Zod validation
- Routes with middleware
```

---

*Document Version: 1.0*  
*Last Updated: 2026-02-11*
