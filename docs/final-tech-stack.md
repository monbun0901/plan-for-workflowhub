# 🎯 WorkflowHub - Final Tech Stack

> **Status:** ✅ Confirmed  
> **Date:** 2026-02-11

---

## Frontend

| Category | Technology | Reason |
|----------|------------|--------|
| **Framework** | Next.js 14+ (App Router) | SSR, SEO, React ecosystem |
| **Language** | TypeScript | Type safety |
| **Styling** | TailwindCSS | Utility-first, rapid development |
| **State Management** | Zustand | Lightweight, simple API |
| **API Client** | Axios | Industry standard |
| **Validation** | Zod | Type-safe schema validation |

---

## Backend

| Category | Technology | Reason |
|----------|------------|--------|
| **Runtime** | Node.js 22+ (`type: module`) | Modern ESM support, performance |
| **Compiler** | **Babel** | Hỗ trợ ES6+, Class properties, và tính tương thích cao |
| **Framework** | Express.js (OOP Style) | Classes, Services, Repositories, JSDoc |
| **Language** | Modern JavaScript (ESM) | Sạch sẽ, chuẩn hóa import/export |
| **Database** | MySQL 8+ | ACID, relational, proven multi-tenant |
| **ORM** | Sequelize | Mature, **CommonJS migrations (.cjs)** |
| **Validation** | Zod | Consistent với Frontend |

---

## AI & Vector

| Category | Technology | Strategy |
|----------|------------|----------|
| **Vector DB** | ChromaDB (v0.5+) | Self-host, port `8001`, persistent |
| **Agent Engine** | **OpenClaw** | Action-oriented, Discord integration |
| **Embeddings** | nomic-embed-text (Ollama) | Local-first, fallback OpenAI |
| **LLM Strategy** | **Hybrid (MVP → Scale)** | |
| **MVP Phase** | OpenAI GPT-4 / Claude | Cloud-based, reliable |
| **Scale Phase** | Ollama (local) + Cloud fallback | Cost optimization |
| **Models (Local)** | llama3.1:8b, qwen2.5:7b | Chat, reasoning |
| **Models (Cloud)** | GPT-4, Claude 3.5 Sonnet | Complex tasks |

---

## Infrastructure

| Category | Technology | Purpose |
|----------|------------|---------|
| **Package Manager** | pnpm | Monorepo, fast, disk efficient |
| **Containerization** | Docker + Docker Compose | Local Stack: MySQL, Redis, Chroma |
| **Admin Tool** | phpMyAdmin | Port `8888`, MySQL GUI |
| **CI/CD** | GitHub Actions | Automation |
| **Caching** | Redis 7 | Sessions, AI buffer, Port `6379` |
| **Storage** | SSD Local (MVP) → S3 (Scale) | Documents, uploads |
| **Queue** | BullMQ (Phase 5+) | Background jobs, workflows |

---

## DevOps & Monitoring

| Category | Technology | Purpose |
|----------|------------|---------|
| **Logging** | Winston | Structured logging |
| **Error Tracking** | Sentry (optional) | Production errors |
| **Health Checks** | Custom `/health` endpoint | Container orchestration |
| **Backup** | MySQL dumps (daily) | Data safety |

---

## Security

| Aspect | Implementation |
|--------|----------------|
| **Authentication** | JWT (Access + Refresh tokens) |
| **Password Hashing** | bcrypt (10 rounds) |
| **Access Token** | 15 minutes lifetime |
| **Refresh Token** | 7 days lifetime |
| **Rate Limiting** | 100 req/15min per IP |
| **CORS** | Whitelist frontend domains |

---

## Development Stack

```
┌────────────────────────────────────────────────────────────┐
│                    WORKFLOWHUB STACK                       │
├────────────────────────────────────────────────────────────┤
│                                                            │
│  FRONTEND (apps/web)                                       │
│  ├── Framework:       Next.js 14+ (App Router)             │
│  ├── Language:        TypeScript                           │
│  ├── Styling:         TailwindCSS                          │
│  ├── State:           Zustand                              │
│  ├── API Client:      Axios                                │
│  └── Validation:      Zod                                  │
│                                                            │
│  BACKEND (apps/api)                                        │
│  ├── Runtime:         Node.js 22+ (ESM)                    │
│  ├── Compiler:        Babel                                │
│  ├── Architecture:    Feature-Based (OOP)                  │
│  ├── Database:        MySQL 8+                             │
│  ├── ORM:             Sequelize (.cjs migrations)          │
│  └── Validation:      Zod                                  │
│                                                            │
│  AI & VECTOR                                               │
│  ├── Vector DB:       ChromaDB (Port 8001)                 │
│  ├── Agent Engine:    OpenClaw (Discord + Web)             │
│  ├── Embeddings:      nomic-embed-text / OpenAI            │
│  ├── LLM (MVP):       OpenAI GPT-4 / Claude 3.5            │
│  └── LLM (Scale):     Ollama (local) + Cloud fallback      │
│                                                            │
│  INFRASTRUCTURE                                            │
│  ├── Package:         pnpm (monorepo)                      │
│  ├── Container:       Docker Compose (Full Stack)          │
│  ├── Admin:           phpMyAdmin (Port 8888)               │
│  ├── Cache:           Redis 7 (Port 6379)                  │
│  └── CI/CD:           GitHub Actions                       │
│                                                            │
└────────────────────────────────────────────────────────────┘
```

---

## Why These Choices?

### MySQL vs MongoDB
- ✅ **ACID Guarantees:** Critical for multi-tenant isolation
- ✅ **Relational Schema:** ERD already designed, clear relationships
- ✅ **Mature Ecosystem:** Sequelize, migrations, tooling
- ✅ **Indexing:** Proven strategy cho tenant isolation

### Sequelize vs Prisma
- ✅ **CLI Migrations:** Mature migration system
- ✅ **Raw SQL:** Flexibility khi cần optimize queries
- ✅ **Ecosystem:** Large community, many plugins

### Zustand vs Redux
- ✅ **Simplicity:** Minimal boilerplate
- ✅ **Performance:** No context re-renders
- ✅ **TypeScript:** First-class support

### Zod vs class-validator
- ✅ **Type Inference:** TypeScript types from schemas
- ✅ **Frontend/Backend:** Same lib, consistent validation
- ✅ **Composability:** Easy schema composition

### Babel + ESM + CJS (Hybrid Architecture)
- ✅ **Modern Code:** Viết ESM sạch sẽ trong `src/`
- ✅ **Stability:** Dùng `.cjs` cho migrations để Sequelize CLI chạy ổn định
- ✅ **Babel:** Đảm bảo transpilation tốt nhất cho các class properties

### Docker Local Stack vs Cloud DB
- ✅ **Zero Latency:** Truy vấn MySQL/Redis/ChromaDB hỏa tốc (1ms)
- ✅ **Zero Cost Storage:** Lưu trữ tri thức AI không giới hạn trên SSD cục bộ
- ✅ **Full Control:** Tự quản lý Version và Configuration (Ports 8001, 8888)

### Chroma vs Pinecone
- ✅ **Cost:** Free, self-hosted
- ✅ **Control:** Full data ownership
- ✅ **Simplicity:** Easy setup, lightweight
- ❌ **Cons:** No managed service (phải self-host)

### AI: Hybrid Strategy
- **MVP:** Cloud (GPT-4/Claude) - reliable, proven
- **Scale:** Local (Ollama) cho simple tasks - cost optimization
- **Fallback:** Cloud for complex reasoning

---

## Migration Path

```
MVP (Phase 1-3)
├── Cloud AI only (OpenAI/Claude)
├── Chroma (single node)
├── Redis (optional)
└── Docker Compose (local dev)

Production (Phase 4+)
├── Hybrid AI (Ollama + Cloud)
├── Chroma (clustering)
├── Redis (required)
├── BullMQ (queue)
└── Docker Swarm / K8s

Scale (Future)
├── Microservices (if needed)
├── Read replicas (MySQL)
├── CDN (Cloudflare)
└── Multi-region (if needed)
```

---

## Confirmed Decisions

- [x] Database: **MySQL 8+** (ACID, relational)
- [x] ORM: **Sequelize** (migrations, raw SQL)
- [x] State: **Zustand** (simplicity)
- [x] Validation: **Zod** (type-safe, cross-platform)
- [x] Vector DB: **Chroma** (self-host, free)
- [x] AI: **Hybrid** (MVP cloud → Scale local+cloud)

---

*Document Version: 1.0*  
*Last Updated: 2026-02-11*
