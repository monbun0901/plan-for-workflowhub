# WorkflowHub - Setup & Common Implementation

**Date:** 2026-02-11  
**Version:** v1  
**Skills:** `senior-architect`, `nodejs-best-practices`, `docker-expert`

---

## 🎯 Overview

Implementation guide cho project setup và common workflows. Đây là bước đầu tiên trước khi implement frontend/backend.

### Skills Map

| Domain | Skills | Files |
|--------|--------|-------|
| **Frontend** | `nextjs-best-practices`, `frontend-developer`, `senior-architect` | [v1-frontend.md](../frontend/v1-frontend.md) |
| **Backend** | `nodejs-best-practices`, `backend-architect`, `backend-security-coder` | [v1-backend.md](../backend/v1-backend.md) |
| **DevOps** | `CI/CD automation`, `devops-deployment`, `docker-expert` | [v1-devops.md](../devops/v1-devops.md) |
| **Database** | `database-architect`, `backend-security-coder` | [v1-database.md](../database/v1-database.md) |

---

## 📦 Prerequisites: Install Skills

### Option A: NPX (Recommended)
```bash
npx antigravity-awesome-skills
```

### Option B: Git Clone
```bash
git clone https://github.com/sickn33/antigravity-awesome-skills.git ~/.agent/skills
```

**Verification:**
```bash
ls ~/.agent/skills/skills/next js-best-practices
ls ~/.agent/skills/skills/nodejs-best-practices
ls ~/.agent/skills/skills/docker-expert
```

---

## Phase 1: Project Setup & Infrastructure

**Duration:** 1-2 tuần  
**Skills:** `senior-architect`, `nodejs-best-practices`, `docker-expert`

### 1.1 Monorepo Setup

#### With `senior-architect` skill:
```
@senior-architect
Tạo monorepo structure cho WorkflowHub với:
- apps/web (Next.js)
- apps/api (Express.js)
- packages/shared (TypeScript types)
```

**Expected Structure:**
```
workflowhub/
├── apps/
│   ├── web/                 # Next.js frontend
│   └── api/                 # Express.js backend
├── packages/
│   └── shared/              # Shared types
├── pnpm-workspace.yaml
├── .gitignore
├── turbo.json              # Turborepo (optional)
└── package.json
```

#### Best Practices Checklist:
- [ ] **Monorepo:** pnpm workspace configured
- [ ] **Versioning:** Consistent package versions
- [ ] **Root scripts:** Dev, build, test commands
- [ ] **.nvmrc:** Node version locked (22+)
- [ ] **.editorconfig:** Consistent formatting

### 1.2 Database Setup (MySQL + Sequelize)

#### With `database-architect` skill:
```
@database-architect
Setup MySQL với Sequelize cho multi-tenant architecture:
- Connection pooling
- Migration strategy
- Indexing plan cho organization_id
```

**sequelize configuration:**
```javascript
// apps/api/config/database.js
module.exports = {
  development: {
    dialect: 'mysql',
    host: '127.0.0.1',
    port: 3306,
    database: 'workflowhub_dev',
    username: 'root',
    password: 'secret',
    pool: {
      max: 10,
      min: 0,
      acquire: 30000,
      idle: 10000,
    },
    logging: console.log,
  },
  production: {
    use_env_variable: 'DATABASE_URL',
    dialect: 'mysql',
    pool: {
      max: 20,
      min: 5,
      acquire: 60000,
      idle: 10000,
    },
    logging: false,
  },
};
```

**Migration naming convention:**
```
YYYYMMDDHHMMSS-descriptive-name.js
```

#### Best Practices Checklist:
- [ ] **Pooling:** Connection pool configured
- [ ] **Migrations:** Sequelize CLI setup
- [ ] **Indexes:** organization_id indexed on all tables
- [ ] **Cascades:** Foreign key cascades defined
- [ ] **Timestamps:** created_at, updated_at automatic

### 1.3 Docker Setup

#### With `docker-expert` skill:
```
@docker-expert
Create production-ready docker-compose.yml cho:
- MySQL 8
- Redis
- API (Express)
- Web (Next.js)
```

**docker-compose.yml:**
```yaml
version: '3.8'

services:
  mysql:
    image: mysql:8
    container_name: workflowhub-mysql
    environment:
      MYSQL_ROOT_PASSWORD: ${DB_ROOT_PASSWORD}
      MYSQL_DATABASE: workflowhub_dev
      MYSQL_USER: ${DB_USER}
      MYSQL_PASSWORD: ${DB_PASSWORD}
    ports:
      - "3306:3306"
    volumes:
      - mysql_data:/var/lib/mysql
    healthcheck:
      test: ["CMD", "mysqladmin", "ping", "-h", "localhost"]
      interval: 10s
      timeout: 5s
      retries: 5

  redis:
    image: redis:7-alpine
    container_name: workflowhub-redis
    ports:
      - "6379:6379"
    volumes:
      - redis_data:/data
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 10s
      timeout: 3s
      retries: 5

  api:
    build:
      context: ./apps/api
      dockerfile: Dockerfile
    container_name: workflowhub-api
    ports:
      - "5000:5000"
    environment:
      NODE_ENV: production
      DATABASE_URL: mysql://${DB_USER}:${DB_PASSWORD}@mysql:3306/workflowhub_dev
      REDIS_URL: redis://redis:6379
      JWT_ACCESS_SECRET: ${JWT_ACCESS_SECRET}
      JWT_REFRESH_SECRET: ${JWT_REFRESH_SECRET}
    depends_on:
      mysql:
        condition: service_healthy
      redis:
        condition: service_healthy

  web:
    build:
      context: ./apps/web
      dockerfile: Dockerfile
    container_name: workflowhub-web
    ports:
      - "3000:3000"
    environment:
      NEXT_PUBLIC_API_URL: http://api:5000
    depends_on:
      - api

volumes:
  mysql_data:
  redis_data:
```

#### Best Practices Checklist:
- [ ] **Health checks:** All services have healthchecks
- [ ] **.dockerignore:** node_modules, .git excluded
- [ ] **Multi-stage builds:** Production images optimized
- [ ] **Environment variables:** All secrets in .env
- [ ] **Volumes:** Data persistence configured

### 1.4 Image Optimization with SlimToolkit

```bash
# After building
docker build -t workflowhub-api:fat ./apps/api

# Slim down (70-90% reduction)
slim build --target workflowhub-api:fat --tag workflowhub-api:slim

# Verify
docker images | grep workflowhub-api
```

---

## Implementation Workflows

### Workflow 1: Feature Development

```bash
# 1. Create feature branch
git checkout -b feature/project-management

# 2. Invoke skills for guidance
@senior-architect @nodejs-best-practices
Design project management module với CRUD operations

# 3. Implement with skill best practices
# - Follow architecture from @backend-architect
# - Apply security from @backend-security-coder

# 4. Test
pnpm run test

# 5. Create PR
git push origin feature/project-management
```

### Workflow 2: Security Audit

```bash
@backend-security-coder
Audit auth module cho vulnerabilities:
- SQL injection
- XSS
- CSRF
- JWT security
```

### Workflow 3: Performance Optimization

```bash
@database-architect
Optimize database queries:
- Identify N+1 queries
- Add indexes
- Review query execution plans
```

---

## Verification & Quality Gates

### Before Each Commit
- [ ] **Linting:** `pnpm run lint` passes
- [ ] **Type Check:** `pnpm run type-check` passes
- [ ] **Tests:** `pnpm run test` passes
- [ ] **Build:** `pnpm run build` succeeds

### Before Each PR
- [ ] **Code Review:** At least 1 approval
- [ ] **CI Passes:** All GitHub Actions green
- [ ] **Test Coverage:** No decrease in coverage
- [ ] **Security:** No new vulnerabilities (Snyk/Dependabot)

### Before Deploy
- [ ] **E2E Tests:** Critical flows tested
- [ ] **Performance:** Load testing passed
- [ ] **Security Scan:** Docker images scanned (Trivy)
- [ ] **Backup:** Database backup verified
- [ ] **Rollback Plan:** Documented

---

## Success Metrics

- [ ] **Code Quality:** ESLint + TypeScript strict mode, no errors
- [ ] **Test Coverage:** > 80% unit test coverage
- [ ] **Performance:** API response < 200ms (p95)
- [ ] **Security:** 0 critical vulnerabilities (Snyk)
- [ ] **Build Time:** CI pipeline < 10 minutes
- [ ] **Docker Images:** < 500MB (after slim)
- [ ] **Uptime:** 99.9% (production)

---

## Related Documents

- [v1-frontend.md](../frontend/v1-frontend.md) - Frontend implementation
- [v1-backend.md](../backend/v1-backend.md) - Backend implementation
- [v1-devops.md](../devops/v1-devops.md) - CI/CD & deployment
- [v1-database.md](../database/v1-database.md) - Database security & optimization

---

*Document Version: 1.0*  
*Last Updated: 2026-02-11*
