# Frontend Implementation Guide

**Version:** v2.0.0
**Date:** 2026-02-11
**Tech Stack:** Next.js 14+ (App Router), Zustand, TailwindCSS, TanStack Query, shadcn/ui, SCSS.

---

## 📋 Overview

Hướng dẫn triển khai Frontend cho WorkflowHub. Tài liệu được tổ chức theo **module** để developer có thể implement từng phần một cách độc lập.

### 🏗️ Component Hierarchy
1. **`ui/` (Atomic)**: shadcn/ui base components (Button, Input, Card...).
2. **`shared/` (Molecules)**: Layout components (Sidebar, Header, DataGrid...).
3. **`features/` (Organisms)**: Domain-specific (ProjectList, IssueTable...).

---

## 🧭 Navigation

### 📐 Architecture & Core
| File | Description | Priority |
|------|-------------|----------|
| [01-architecture.md](01-architecture.md) | Next.js App Router & Routing | P0 |
| [02-state-management.md](02-state-management.md) | Store Slicing & SoC Philosophy | P0 |
| [03-api-service.md](03-api-service.md) | Axios Interceptors & Service Layer | P0 |
| [05-hooks.md](05-hooks.md) | Custom Hooks & Data Fetching | P1 |
| [08-frontend-standards.md](08-frontend-standards.md) | **Coding Rules, SCSS, REM, l10n** | P0 |

### 🏛️ Layouts
| File | Description |
|------|-------------|
| [layouts/01-shell-layout.md](layouts/01-shell-layout.md) | Dashboard Shell (Sidebar + Header + Main) |
| [layouts/02-auth-layout.md](layouts/02-auth-layout.md) | Auth Pages (Login, Register) |
| [layouts/03-crud-page-layout.md](layouts/03-crud-page-layout.md) | Page-based CRUD (Anti-Modal) |

### 🧩 Components (Reusable UI)
| File | Description |
|------|-------------|
| [components/01-sidebar.md](components/01-sidebar.md) | Triple-state sidebar (Expand/Collapse/Overlay) |
| [components/02-header.md](components/02-header.md) | Contextual header with breadcrumbs + search |
| [components/03-navigation.md](components/03-navigation.md) | Data-driven navigation (config-based) |
| [components/04-toast-system.md](components/04-toast-system.md) | Non-blocking async notifications |
| [components/05-data-grid.md](components/05-data-grid.md) | Grid/Table pattern + advanced filters |
| [components/06-form-patterns.md](components/06-form-patterns.md) | Hook-driven form validation |
| [components/07-page-header.md](components/07-page-header.md) | Page header with title, actions, breadcrumbs |
| [components/08-tabs.md](components/08-tabs.md) | Tabs navigation (underline/pills variants) |
| [components/09-form-card.md](components/09-form-card.md) | Card wrapper for CRUD forms |
| [components/10-breadcrumbs.md](components/10-breadcrumbs.md) | Navigation breadcrumbs |
| [components/11-actions-bar.md](components/11-actions-bar.md) | Search + filters + actions for list pages |
| [components/12-dropdown-filter.md](components/12-dropdown-filter.md) | Single/multi-select dropdown filter |UX |

### 📦 Constants
| File | Description |
|------|-------------|
| [constants/01-navigation-config.md](constants/01-navigation-config.md) | Sidebar & Header Menu Config |
| [constants/02-api-endpoints.md](constants/02-api-endpoints.md) | API Route Constants |
| [constants/03-app-routes.md](constants/03-app-routes.md) | Frontend Route Paths |
| [constants/04-ui-tokens.md](constants/04-ui-tokens.md) | Status Labels, Roles, Priorities |

### 🗄️ Stores (Zustand)
| File | Description |
|------|-------------|
| [stores/01-auth-store.md](stores/01-auth-store.md) | useAuthStore (Identity) |
| [stores/02-layout-store.md](stores/02-layout-store.md) | useLayoutStore (Sidebar, Overlays) |
| [stores/03-config-store.md](stores/03-config-store.md) | useConfigStore (Theme, Language) |
| [stores/04-tenant-store.md](stores/04-tenant-store.md) | useTenantStore (Org Context) |

### 🔀 Cross-cutting Patterns
| File | Description |
|------|-------------|
| [patterns/01-localization.md](patterns/01-localization.md) | l10n-ready Pattern & Dictionary |
| [patterns/02-resilient-ui.md](patterns/02-resilient-ui.md) | Empty State, Skeleton, Error Boundary |

### 📑 Pages (UI/UX per Page)
| File | Description |
|------|-------------|
| [pages/README.md](pages/README.md) | **Full Page Map** — 26 pages, wireframes, features, status |

### 🚀 Scale-up & Future
| File | Description | Note |
|------|-------------|------|
| [06-best-practices.md](06-best-practices.md) | Performance & Accessibility | Scale-up |
| [07-ai-integration.md](07-ai-integration.md) | AI Chat & Workflow UI | Phase 2 |

---

## 📚 Related Documents

- [../backend/README.md](../backend/README.md) - API Documentation
- [../../roadmap/README.md](../../roadmap/README.md) - Project Roadmap

---

*Last Updated: 2026-02-11*
