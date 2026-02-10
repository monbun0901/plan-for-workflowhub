# WorkflowHub - DevOps & CI/CD Implementation

**Date:** 2026-02-11  
**Version:** v1  
**Skills:** `CI/CD automation`, `devops-deployment`, `docker-expert`

---

## 🎯 Overview

CI/CD và deployment guide cho WorkflowHub sử dụng GitHub Actions và Docker.

---

## Phase 5: CI/CD & Deployment

**Duration:** Setup once, iterate throughout  
**Skills:** `CI/CD automation`, `devops-deployment`

### 5.1 GitHub Actions Workflows

#### With `CI/CD automation` skill:
```
@CI/CD automation
Create GitHub Actions workflows:
- Lint & Type check
- Unit tests
- Build verification
- Docker image build
- Deploy to staging/production
```

**.github/workflows/ci.yml:**
```yaml
name: CI

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main, develop]

jobs:
  lint-and-type-check:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '22'
          cache: 'pnpm'
      
      - name: Install pnpm
        run: npm install -g pnpm
      
      - name: Install dependencies
        run: pnpm install --frozen-lockfile
      
      - name: Lint
        run: pnpm run lint
      
      - name: Type check
        run: pnpm run type-check

  unit-tests:
    runs-on: ubuntu-latest
    needs: lint-and-type-check
    
    services:
      mysql:
        image: mysql:8
        env:
          MYSQL_ROOT_PASSWORD: test
          MYSQL_DATABASE: workflowhub_test
        ports:
          - 3306:3306
        options: >-
          --health-cmd="mysqladmin ping"
          --health-interval=10s
          --health-timeout=5s
          --health-retries=3
      
      redis:
        image: redis:7-alpine
        ports:
          - 6379:6379
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '22'
          cache: 'pnpm'
      
      - name: Install pnpm
        run: npm install -g pnpm
      
      - name: Install dependencies
        run: pnpm install --frozen-lockfile
      
      - name: Run tests
        run: pnpm run test
        env:
          DATABASE_URL: mysql://root:test@127.0.0.1:3306/workflowhub_test
          REDIS_URL: redis://127.0.0.1:6379
      
      - name: Upload coverage
        uses: codecov/codecov-action@v3
        if: always()

  build-check:
    runs-on: ubuntu-latest
    needs: lint-and-type-check
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '22'
          cache: 'pnpm'
      
      - name: Install pnpm
        run: npm install -g pnpm
      
      - name: Install dependencies
        run: pnpm install --frozen-lockfile
      
      - name: Build frontend
        run: pnpm --filter web run build
      
      - name: Build backend
        run: pnpm --filter api run build

  docker-build:
    runs-on: ubuntu-latest
    needs: [unit-tests, build-check]
    if: github.event_name == 'push' && github.ref == 'refs/heads/main'
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3
      
      - name: Login to GitHub Container Registry
        uses: docker/login-action@v3
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}
      
      - name: Build and push API image
        uses: docker/build-push-action@v5
        with:
          context: ./apps/api
          push: true
          tags: |
            ghcr.io/${{ github.repository }}/api:latest
            ghcr.io/${{ github.repository }}/api:${{ github.sha }}
          cache-from: type=gha
          cache-to: type=gha,mode=max
      
      - name: Build and push Web image
        uses: docker/build-push-action@v5
        with:
          context: ./apps/web
          push: true
          tags: |
            ghcr.io/${{ github.repository }}/web:latest
            ghcr.io/${{ github.repository }}/web:${{ github.sha }}
          cache-from: type=gha
          cache-to: type=gha,mode=max
```

#### CI/CD Best Practices Checklist:
- [ ] **Branch Protection:** Main branch requires PR + reviews
- [ ] **Status Checks:** All checks must pass before merge
- [ ] **Caching:** Dependencies cached (pnpm, Docker layers)
- [ ] **Secrets:** All credentials in GitHub Secrets
- [ ] **Versioning:** Git SHA tags on Docker images

### 5.2 Dockerfile Best Practices

#### API Dockerfile (Multi-stage)
```dockerfile
# apps/api/Dockerfile
FROM node:22-alpine AS base
RUN npm install -g pnpm

# Dependencies stage
FROM base AS dependencies
WORKDIR /app
COPY package.json pnpm-lock.yaml pnpm-workspace.yaml ./
COPY apps/api/package.json ./apps/api/
COPY packages/shared/package.json ./packages/shared/
RUN pnpm install --frozen-lockfile

# Build stage
FROM base AS build
WORKDIR /app
COPY --from=dependencies /app/node_modules ./node_modules
COPY . .
RUN pnpm --filter api run build

# Production stage
FROM node:22-alpine AS production
WORKDIR /app

# Install production dependencies only
COPY package.json pnpm-lock.yaml pnpm-workspace.yaml ./
COPY apps/api/package.json ./apps/api/
COPY packages/shared/package.json ./packages/shared/
RUN npm install -g pnpm && \
    pnpm install --frozen-lockfile --prod && \
    pnpm store prune

# Copy built files
COPY --from=build /app/apps/api/dist ./apps/api/dist
COPY --from=build /app/packages/shared/dist ./packages/shared/dist

EXPOSE 5000

CMD ["node", "apps/api/dist/index.js"]
```

#### Web Dockerfile (Next.js)
```dockerfile
# apps/web/Dockerfile
FROM node:22-alpine AS base
RUN npm install -g pnpm

# Dependencies stage
FROM base AS dependencies
WORKDIR /app
COPY package.json pnpm-lock.yaml pnpm-workspace.yaml ./
COPY apps/web/package.json ./apps/web/
COPY packages/shared/package.json ./packages/shared/
RUN pnpm install --frozen-lockfile

# Build stage
FROM base AS build
WORKDIR /app
COPY --from=dependencies /app/node_modules ./node_modules
COPY . .
ENV NEXT_TELEMETRY_DISABLED 1
RUN pnpm --filter web run build

# Production stage
FROM node:22-alpine AS production
WORKDIR /app

ENV NODE_ENV production
ENV NEXT_TELEMETRY_DISABLED 1

RUN addgroup --system --gid 1001 nodejs
RUN adduser --system --uid 1001 nextjs

COPY --from=build /app/apps/web/public ./public
COPY --from=build --chown=nextjs:nodejs /app/apps/web/.next/standalone ./
COPY --from=build --chown=nextjs:nodejs /app/apps/web/.next/static ./.next/static

USER nextjs

EXPOSE 3000

ENV PORT 3000

CMD ["node", "server.js"]
```

### 5.3 Docker Compose for Local Development

```yaml
# docker-compose.dev.yml
version: '3.8'

services:
  mysql:
    image: mysql:8
    container_name: workflowhub-mysql-dev
    environment:
      MYSQL_ROOT_PASSWORD: root
      MYSQL_DATABASE: workflowhub_dev
    ports:
      - "3306:3306"
    volumes:
      - mysql_dev_data:/var/lib/mysql
    healthcheck:
      test: ["CMD", "mysqladmin", "ping", "-h", "localhost"]
      interval: 5s
      timeout: 3s
      retries: 5

  redis:
    image: redis:7-alpine
    container_name: workflowhub-redis-dev
    ports:
      - "6379:6379"
    volumes:
      - redis_dev_data:/data

  api:
    build:
      context: .
      dockerfile: apps/api/Dockerfile.dev
    container_name: workflowhub-api-dev
    ports:
      - "5000:5000"
    environment:
      NODE_ENV: development
      DATABASE_URL: mysql://root:root@mysql:3306/workflowhub_dev
      REDIS_URL: redis://redis:6379
    volumes:
      - ./apps/api/src:/app/apps/api/src
      - ./packages/shared/src:/app/packages/shared/src
    depends_on:
      mysql:
        condition: service_healthy
      redis:
        condition: service_started
    command: pnpm --filter api run dev

  web:
    build:
      context: .
      dockerfile: apps/web/Dockerfile.dev
    container_name: workflowhub-web-dev
    ports:
      - "3000:3000"
    environment:
      NEXT_PUBLIC_API_URL: http://localhost:5000
    volumes:
      - ./apps/web/src:/app/apps/web/src
      - ./packages/shared/src:/app/packages/shared/src
    depends_on:
      - api
    command: pnpm --filter web run dev

volumes:
  mysql_dev_data:
  redis_dev_data:
```

### 5.4 Deployment Strategies

#### Option 1: Railway / Render
```bash
# Railway CLI
railway link
railway up

# Environment variables
railway variables set DATABASE_URL="mysql://..."
railway variables set REDIS_URL="redis://..."
railway variables set JWT_ACCESS_SECRET="..."
```

#### Option 2: VPS (Docker Compose)
```bash
# SSH into VPS
ssh user@your-vps-ip

# Clone project
git clone https://github.com/your-username/workflowhub.git
cd workflowhub

# Create .env file
cp .env.example .env.production
nano .env.production  # Edit with production values

# Deploy
docker-compose -f docker-compose.prod.yml up -d

# View logs
docker-compose logs -f api
```

#### Option 3: Kubernetes (Future)
```yaml
# k8s/api-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: workflowhub-api
spec:
  replicas: 3
  selector:
    matchLabels:
      app: workflowhub-api
  template:
    metadata:
      labels:
        app: workflowhub-api
    spec:
      containers:
      - name: api
        image: ghcr.io/your-username/workflowhub/api:latest
        ports:
        - containerPort: 5000
        env:
        - name: DATABASE_URL
          valueFrom:
            secretKeyRef:
              name: workflowhub-secrets
              key: database-url
```

---

## Skill Usage Examples

### Example 1: Add New CI Step
```
@CI/CD automation
Add security scanning to CI pipeline:
- Snyk vulnerability scan
- Trivy Docker image scan
- SAST with SonarQube
```

### Example 2: Optimize Docker Build
```
@docker-expert
Optimize Docker images:
- Multi-stage builds
- Layer caching
- .dockerignore
- Alpine base images
```

### Example 3: Setup Monitoring
```
@devops-deployment
Setup production monitoring:
- Health check endpoints
- Prometheus metrics
- Grafana dashboards
- Alert rules
```

---

## Related Documents

- [v1-setup-and-workflows.md](../common/v1-setup-and-workflows.md) - Docker setup basics
- [v1-backend.md](../backend/v1-backend.md) - Backend API
- [v1-frontend.md](../frontend/v1-frontend.md) - Frontend build

---

*Document Version: 1.0*  
*Last Updated: 2026-02-11*
