# Database Tables Overview

**Version:** v1  
**Date:** 2026-02-11  
**Database:** MySQL 8.0+  
**ORM:** Sequelize

---

## 📋 Table of Contents

### 🟢 Phase 1: Core System (MVP)

#### 📅 Master Data (Danh mục dùng chung)
1. [users.md](tables/users.md) - User accounts
2. [organizations.md](tables/organizations.md) - Multi-tenant root
3. [members.md](tables/members.md) - Org memberships
4. [categories.md](tables/categories.md) - **NEW:** Dynamic Categories
5. [workflow_statuses.md](tables/workflow_statuses.md) - **NEW:** Custom Workflows
6. [tags.md](tables/tags.md) - **NEW:** Global Tags
7. [roles.md](tables/roles.md) - **NEW:** Dynamic Roles & Permissions
8. [visibility_levels.md](tables/visibility_levels.md) - **NEW:** Visibility (Access Levels)

#### 📈 Transactional Data (Dữ liệu nghiệp vụ)
9. [projects.md](tables/projects.md) - Projects (Updated)
10. [project_members.md](tables/project_members.md) - **NEW:** Project Assignees
11. [project_tags.md](tables/project_tags.md) - **NEW:** Project Tags
12. [issues.md](tables/issues.md) - Issues tracking (Updated)
13. [issue_categories.md](tables/issue_categories.md) - **NEW:** Issue Categories
14. [issue_assignees.md](tables/issue_assignees.md) - **NEW:** Issue Assignees
15. [issue_tags.md](tables/issue_tags.md) - **NEW:** Issue Tags
16. [tasks.md](tables/tasks.md) - Task management (Updated)
17. [task_categories.md](tables/task_categories.md) - **NEW:** Task Categories
18. [task_assignees.md](tables/task_assignees.md) - **NEW:** Task Assignees
19. [task_tags.md](tables/task_tags.md) - **NEW:** Task Tags
20. [task_assignments.md](tables/task_assignments.md) - Assignment history
21. [v1-database-security.md](v1-database-security.md) - Security & optimization
22. [v1-redis-strategy.md](v1-redis-strategy.md) - **NEW:** Caching & Performance
23. [../devops/v1-infrastructure-docker.md](../devops/v1-infrastructure-docker.md) - **NEW:** Infrastructure (Docker Stack)

### 🚀 Phase 2: Scale-up (AI & Automation)
24. [documents.md](tables/documents.md) - Knowledge base (Scale-up)
25. [document_categories.md](tables/document_categories.md) - **NEW:** Doc Categories
26. [document_collaborators.md](tables/document_collaborators.md) - **NEW:** Doc Collaborators
27. [document_tags.md](tables/document_tags.md) - **NEW:** Document Tags
28. [agents.md](tables/agents.md) - AI agent configs
29. [chats.md](chats.md) - Chat sessions
30. [messages.md](messages.md) - Chat messages + Tokens
31. [workflow_templates.md](workflow_templates.md) - Automation logic
32. [workflow_instances.md](workflow_instances.md) - Automation runs

---

## 🔑 Key Principles

### Multi-Tenant Isolation

**Every table (except `users`) MUST have `organization_id`:**

```sql
organization_id VARCHAR(36) NOT NULL REFERENCES organizations(id)
```

**Every query MUST filter by `organization_id`:**

```typescript
where: { 
  organization_id: organizationId,  // CRITICAL
  // ... other filters
}
```

### Indexing Strategy

```sql
-- Primary tenant filter
CREATE INDEX idx_{table}_org_id ON {table}(organization_id);

-- Composite indexes for common queries
CREATE INDEX idx_{table}_org_status ON {table}(organization_id, status);
```

---

## 📊 Entity Relationship

```
users (1) ──── (N) members (N) ──── (1) organizations
                                              │
                                              │ (1:N)
                                              ▼
                                          projects
                                              │
                         ┌────────────────────┼────────────────────┐
                         │                    │                    │
                         ▼                    ▼                    ▼
                      issues              documents            workflows
                         │
                         ▼
                      tasks
```

---

## 🛡️ Security Checklist

- [ ] All tables (except users) have `organization_id`
- [ ] All queries filter by `organization_id`
- [ ] Composite indexes include `organization_id`
- [ ] Foreign keys use ON DELETE CASCADE
- [ ] Sensitive data encrypted (see [v1-database-security.md](v1-database-security.md))

---

## 📚 Related Documents

- [../backend/03-multi-tenant.md](../backend/03-multi-tenant.md) - Tenant isolation patterns
- [../backend/04-projects-module.md](../backend/04-projects-module.md) - Repository examples
- [../../basics/step-5-data-design.md](../../basics/step-5-data-design.md) - Full data design

---

*Last Updated: 2026-02-11*
