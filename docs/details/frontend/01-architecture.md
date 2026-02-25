# Frontend Architecture & Structure

**Version:** v1  
**Skills:** `nextjs-best-practices`, `senior-architect`

---

## 🏗️ Next.js 14+ App Router Structure

Chúng ta sử dụng cấu trúc thư mục tối ưu cho khả năng mở rộng và cô lập các route group.

### Directory Tree

```
apps/web/src/
├── app/
│   ├── (auth)/                  # Route group: Auth pages
│   │   ├── login/page.tsx
│   │   ├── register/page.tsx
│   │   └── layout.tsx           # Auth-specific layout
│   │
│   ├── (dashboard)/             # Route group: Main App
│   │   ├── projects/
│   │   │   ├── page.tsx         # /projects (List)
│   │   │   └── [id]/            # /projects/[id] (Detail)
│   │   │       ├── issues/
│   │   │       ├── tasks/
│   │   │       └── documents/
│   │   ├── chat/                # (Phase 2)
│   │   └── layout.tsx           # Sidebar & Header layout
│   │
│   ├── layout.tsx               # Global providers & HTML tag
│   └── page.tsx                 # Landing page
│
├── components/
│   ├── ui/                      # shadcn/ui base (reusable)
│   ├── features/                # Domain-specific components
│   │   ├── projects/            # ProjectCard, ProjectList
│   │   ├── issues/
│   │   └── tasks/
│   └── shared/                  # Common components (layout elements)
│
├── app/
│   ├── contexts/                # Context layer (page-level logic)
│   │   ├── shared/              # Reusable fetch contexts
│   │   │   ├── CategoryFetchContext.tsx
│   │   │   ├── MemberFetchContext.tsx
│   │   │   └── StatusFetchContext.tsx
│   │   ├── tasks/               # Task feature contexts
│   │   │   ├── TaskListContext.tsx
│   │   │   └── TaskFormContext.tsx
│   │   ├── issues/              # Issue feature contexts (planned)
│   │   ├── documents/           # Document feature contexts (planned)
│   │   └── index.ts             # Barrel export
│   └── ...
│
├── hooks/                       # Custom React hooks
├── services/                    # API Service layer
├── stores/                      # Zustand state stores (data layer)
├── types/                       # Global TypeScript definitions
└── utils/                       # Helper functions
```

### Context Layer (`app/contexts/`)

Contexts wrap Zustand stores to provide page-level logic separation. They do **not** replace stores — stores remain the source of truth for data.

**Pattern:**
- **Shared contexts** (`shared/`): Auto-fetch wrappers (e.g., `CategoryFetchProvider` fetches categories on mount)
- **Feature contexts** (`tasks/`, `issues/`): Encapsulate filtering, sorting, pagination, event handlers per feature
- **Pages** consume contexts via custom hooks (`useTaskListContext()`) and only contain UI rendering

---

## 🚦 Server vs Client Components

**Nguyên tắc vàng:** `Fetch data on Server, Interact on Client.`

### 🟩 Server Components (Default)
- **Dùng cho:** Data fetching từ API, Layouts, Static content.
- **Lợi ích:** SEO tốt hơn, JS bundle nhỏ hơn, Bảo mật API keys.

### 🟦 Client Components (`'use client'`)
- **Dùng cho:** Event listeners (click, change), Forms, Browser APIs, Local state (useState).
- **Lợi ích:** Tương tác mượt mà, Real-time updates.

---

## 📦 Component Composition Pattern

```tsx
// @follows senior-architect: Composition pattern
export default async function ProjectPage({ params }: { params: { id: string } }) {
  // 1. Fetch data on SERVER
  const project = await getProject(params.id);

  return (
    <main>
      <header>
        <h1>{project.name}</h1>
        {/* 2. Pass data to CLIENT components for interaction */}
        <EditProjectButton project={project} />
      </header>
      
      <section>
        <Sidebar initialData={project.items} />
      </section>
    </main>
  );
}
```

---

## 🧭 Loading & Error Strategy

Mỗi route folder nên có:
- `loading.tsx`: Skeleton screens hiển thị khi server đang xử lý.
- `error.tsx`: Catch errors và hiển thị UI phục hồi cho user.

---

## ✅ Best Practices Checklist

- [ ] Sử dụng **Route Groups** `()` để tổ chức SEO & Layouts.
- [ ] Tận dụng **Dynamic Parameters** `[id]` cho các trang chi tiết.
- [ ] Luôn đặt metadata API trong `page.tsx`.
- [ ] Tránh lạm dụng `'use client'` ở root của trang.

---

## 📚 Related Documents

- [../README.md](../README.md) - Go back to navigation
- [04-components.md](04-components.md) - Detail on component design
- [../../basics/step-3-architecture.md](../../basics/step-3-architecture.md) - System architecture

---

*Last Updated: 2026-02-11*
