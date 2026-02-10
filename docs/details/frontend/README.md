# Frontend Implementation Guide

**Version:** v1  
**Date:** 2026-02-11  
**Tech Stack:** Next.js 14+ (App Router), Zustand, TailwindCSS, TanStack Query, shadcn/ui.

---

## 📋 Overview

Hướng dẫn triển khai Frontend cho WorkflowHub. Chúng ta sử dụng Next.js App Router làm nền tảng, kết hợp với các library hiện đại để đảm bảo hiệu năng và khả năng mở rộng.

---

## 🧭 Navigation

### 🟢 Phase 1: Core Essentials
| File | Description | Priority |
|------|-------------|----------|
| [01-architecture.md](01-architecture.md) | Next.js structure & Routing | P0 |
| [02-state-management.md](02-state-management.md) | Global state with Zustand | P0 |
| [03-api-service.md](03-api-service.md) | API clients & Interceptors | P0 |
| [04-components.md](04-components.md) | Component architecture & UI Kit | P1 |
| [05-hooks.md](05-hooks.md) | Custom hooks & Data fetching | P1 |

### 🚀 Phase 2: Scale-up & Optimization
| File | Description | Note |
|------|-------------|------|
| [06-best-practices.md](06-best-practices.md) | Performance & Accessibility | Scale-up |
| [07-ai-integration.md](07-ai-integration.md) | AI Chat & Workflow UI | Phase 2 |
| [08-frontend-standards.md](08-frontend-standards.md) | **NEW:** Coding & Style Standards | P0 |
| [09-ui-ux-patterns.md](09-ui-ux-patterns.md) | **NEW:** Responsive & Shell Patterns | P0 |

---

## 📐 Main Architecture

### 1. App Router
- **(auth)**: Route group cho login, register.
- **(dashboard)**: Route group cho main application (projects, issues, tasks).
- **Server Components**: Mặc định sử dụng cho data fetching.
- **Client Components**: Chỉ sử dụng khi cần tương tác (form, events).

### 2. State Management
- **Zustand**: Quản lý Global State (Auth, UI, Tenant).
- **TanStack Query**: Quản lý Server State (Caching, Sync, Pagination).

---

## 🔧 Skills Usage

### New Page Creation
```
@nextjs-best-practices
Create {feature} page với App Router:
- Server Component cho initial data
- Client Component cho interactive elements
- Loading & Error handlers
```

### Component Design
```
@frontend-developer @senior-architect
Design {component} following project patterns:
- TailwindCSS styling
- shaden/ui base
- JSDoc documentation
```

---

## 📚 Related Documents

- [../../basics/step-3-architecture.md](../../basics/step-3-architecture.md) - High-level architecture
- [../backend/README.md](../backend/README.md) - API Documentation
- [../common/v1-setup-and-workflows.md](../common/v1-setup-and-workflows.md) - Setup guide

---

*Last Updated: 2026-02-11*
