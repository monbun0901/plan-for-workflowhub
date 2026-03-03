# 🚀 Hướng Dẫn Cài Đặt — WorkflowHub

> **Version:** 1.0.0 · **Cập nhật:** 2026-03-03

## Mục Lục

- [1. Yêu Cầu Hệ Thống](#1-yêu-cầu-hệ-thống)
- [2. Cài Đặt Nhanh](#2-cài-đặt-nhanh)
- [3. Biến Môi Trường](#3-biến-môi-trường)
- [4. Docker Services](#4-docker-services)
- [5. Database Setup](#5-database-setup)
- [6. Chạy Development](#6-chạy-development)
- [7. Scripts Reference](#7-scripts-reference)
- [8. Troubleshooting](#8-troubleshooting)

---

## 1. Yêu Cầu Hệ Thống

| Phần mềm | Phiên bản | Ghi chú |
|-----------|----------|---------|
| **Node.js** | ≥ 22.0.0 | Xem `.nvmrc` |
| **pnpm** | ≥ 8.0.0 | Package manager |
| **Docker** | Latest | Cho MySQL, Redis, ChromaDB |
| **Docker Compose** | v2+ | Orchestration |
| **Git** | Latest | Version control |

---

## 2. Cài Đặt Nhanh

```bash
# 1. Clone repository
git clone <repo-url> workflowhub
cd workflowhub

# 2. Cài đặt dependencies
pnpm install

# 3. Cấu hình environment
cp .env.example .env
cp apps/api/.env.example apps/api/.env

# 4. Khởi động Docker services (MySQL, Redis, ChromaDB, phpMyAdmin)
pnpm docker:up

# 5. Chạy database migrations
pnpm db:migrate

# 6. Seed dữ liệu mẫu (optional)
pnpm db:seed

# 7. Khởi động development servers
pnpm dev
```

Sau khi hoàn tất:
- **Frontend:** http://localhost:3000
- **Backend API:** http://localhost:5000
- **phpMyAdmin:** http://localhost:8888

---

## 3. Biến Môi Trường

### 3.1. Root `.env` (Docker Compose)

| Biến | Mặc định | Mô tả |
|------|---------|-------|
| `DB_ROOT_PASSWORD` | `rootpassword` | MySQL root password |
| `DB_NAME` | `workflowhub_db` | Tên database |
| `DB_USER` | `wh_user` | MySQL user |
| `DB_PASSWORD` | `wh_password` | MySQL password |
| `DB_HOST` | `127.0.0.1` | MySQL host |
| `DB_PORT` | `3306` | MySQL port |
| `REDIS_HOST` | `127.0.0.1` | Redis host |
| `REDIS_PORT` | `6379` | Redis port |
| `CHROMA_HOST` | `127.0.0.1` | ChromaDB host |
| `CHROMA_PORT` | `8001` | ChromaDB port |
| `JWT_ACCESS_SECRET` | `your-access-secret-...` | JWT access token secret |
| `JWT_REFRESH_SECRET` | `your-refresh-secret-...` | JWT refresh token secret |
| `JWT_ACCESS_EXPIRY` | `15m` | Access token TTL |
| `JWT_REFRESH_EXPIRY` | `7d` | Refresh token TTL |
| `API_PORT` | `5000` | Backend API port |
| `API_HOST` | `0.0.0.0` | Backend bind address |
| `NODE_ENV` | `development` | Environment |
| `NEXT_PUBLIC_API_URL` | `http://localhost:5000/api/v1` | Frontend API base URL |
| `OPENAI_API_KEY` | *(optional)* | OpenAI API key cho AI module |
| `OLLAMA_HOST` | *(optional)* | Ollama local URL |

### 3.2. `apps/api/.env` (Backend riêng)

| Biến | Mặc định | Mô tả |
|------|---------|-------|
| `CORS_ORIGIN` | `http://localhost:3000` | Allowed frontend origin |
| `LOG_LEVEL` | `debug` | Winston log level |
| `CLOUDINARY_CLOUD_NAME` | `your_cloud_name` | Cloudinary cloud name |
| `CLOUDINARY_API_KEY` | `your_api_key` | Cloudinary API key |
| `CLOUDINARY_API_SECRET` | `your_api_secret` | Cloudinary API secret |

> ⚠️ **Quan trọng:** Thay đổi `JWT_ACCESS_SECRET` và `JWT_REFRESH_SECRET` khi deploy production.

---

## 4. Docker Services

```bash
# Khởi động tất cả services
pnpm docker:up

# Dừng tất cả services
pnpm docker:down

# Xem logs
pnpm docker:logs
```

| Service | Container | Port | URL | Mô tả |
|---------|-----------|------|-----|-------|
| **MySQL** | `wh-mysql` | 3306 | `mysql://localhost:3306` | Primary database |
| **Redis** | `wh-redis` | 6379 | `redis://localhost:6379` | Cache & sessions |
| **ChromaDB** | `wh-chromadb` | 8001 | `http://localhost:8001` | Vector database |
| **phpMyAdmin** | `wh-phpmyadmin` | 8888 | `http://localhost:8888` | DB management UI |

### Volumes (persistent data)

| Volume | Mount | Dùng cho |
|--------|-------|---------|
| `mysql_data` | `/var/lib/mysql` | MySQL data |
| `redis_data` | `/data` | Redis persistence |
| `chromadb_data` | `/chroma/chroma` | ChromaDB embeddings |

---

## 5. Database Setup

### 5.1. Migrations

```bash
# Chạy tất cả migrations
pnpm db:migrate

# Undo migration cuối cùng
pnpm db:migrate:undo
```

Migration history (thứ tự thực thi):

| # | Migration | Nội dung |
|---|----------|---------|
| 1 | `t1-lookup-tables` | Roles, priorities, statuses |
| 2 | `t1-categories` | Category types (6 loại) |
| 3 | `t2-users` | Users table |
| 4 | `t3-projects` | Projects table |
| 5 | `t4-agents` | AI agents table |
| 6 | `t4-core-entities` | Tasks, issues, documents |
| 7 | `t5-operations` | Workflows, chats, messages |
| 8 | `t6-junctions-and-details` | Junction tables (N:M) |
| 9 | `t7-project-members` | Project members + roles |
| 10 | `update-agents-v3` | Agent schema updates |
| 11 | `update-workflows-v3` | Workflow schema updates |
| 12 | `update-chats-v3` | Chat schema updates |
| 13 | `add-api-key-to-agents` | Agent API key field |

### 5.2. Seeders

```bash
# Chạy tất cả seeders
pnpm db:seed
```

| # | Seeder | Dữ liệu |
|---|--------|---------|
| 1 | `seed-master-data` | Roles, statuses, categories mặc định |
| 2 | `seed-users` | Admin, manager, user accounts |
| 3 | `seed-projects` | Sample projects |
| 4 | `seed-issues` | Sample issues |
| 5 | `seed-tasks` | Sample tasks |
| 6 | `seed-knowledge-base` | Sample documents cho AI |

---

## 6. Chạy Development

```bash
# Chạy cả frontend + backend (parallel)
pnpm dev

# Chỉ chạy backend (port 5000)
pnpm dev:api

# Chỉ chạy frontend (port 3000)
pnpm dev:web
```

### Health Check

```bash
curl http://localhost:5000/health
```

Response:
```json
{
  "success": true,
  "message": "API is running",
  "data": {
    "status": "healthy",
    "uptime": 123.456,
    "environment": "development",
    "timestamp": "2026-03-03T10:00:00.000Z"
  }
}
```

### Default Accounts (sau khi seed)

| Role | Email | Password |
|------|-------|----------|
| Admin | *(xem seeder)* | *(xem seeder)* |
| Manager | *(xem seeder)* | *(xem seeder)* |
| User | *(xem seeder)* | *(xem seeder)* |

> Kiểm tra file `apps/api/src/database/seeders/20260215000002-seed-users.cjs` để xem thông tin đăng nhập.

---

## 7. Scripts Reference

| Script | Lệnh | Mô tả |
|--------|------|-------|
| `pnpm dev` | `pnpm --parallel -r run dev` | Start all dev servers |
| `pnpm dev:api` | `pnpm --filter @workflowhub/api run dev` | Start backend only |
| `pnpm dev:web` | `pnpm --filter @workflowhub/web run dev` | Start frontend only |
| `pnpm build` | `pnpm -r run build` | Build all packages |
| `pnpm build:api` | `pnpm --filter @workflowhub/api run build` | Build backend |
| `pnpm build:web` | `pnpm --filter @workflowhub/web run build` | Build frontend |
| `pnpm lint` | `pnpm -r run lint` | Lint all packages |
| `pnpm clean` | `rm -rf node_modules dist .next` | Clean all builds |
| `pnpm docker:up` | `docker compose up -d` | Start Docker services |
| `pnpm docker:down` | `docker compose down` | Stop Docker services |
| `pnpm docker:logs` | `docker compose logs -f` | View Docker logs |
| `pnpm db:migrate` | Sequelize CLI migrate | Run database migrations |
| `pnpm db:migrate:undo` | Sequelize CLI migrate:undo | Undo last migration |
| `pnpm db:seed` | Sequelize CLI db:seed:all | Seed database |

---

## 8. Troubleshooting

### MySQL không connect được
```bash
# Kiểm tra MySQL container
docker ps | grep wh-mysql
docker logs wh-mysql

# Đợi healthcheck pass (10s interval)
docker inspect wh-mysql --format='{{.State.Health.Status}}'
```

### Port conflict
```bash
# Kiểm tra port đang dùng
netstat -ano | findstr :3306
netstat -ano | findstr :5000
```

### Reset hoàn toàn database
```bash
pnpm docker:down
docker volume rm workflowhub_mysql_data
pnpm docker:up
# Đợi MySQL healthy (~30s)
pnpm db:migrate
pnpm db:seed
```

### pnpm install lỗi native modules
```bash
# Nếu bcrypt hoặc sharp lỗi
pnpm install --force
```

---

> **Xem thêm:**
> - [01 — Architecture Overview](./01-architecture-overview.md)
> - [03 — Database Schema](./03-database-schema.md)
