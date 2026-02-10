# Component Architecture

**Version:** v1  
**Skills:** `frontend-developer`, `senior-architect`

---

## 🏗️ Hierarchy & Organization

Tất cả components được phân loại theo mức độ tái sử dụng:

1. **`ui/` (Atomic)**: Các component cơ bản như Button, Input, Modal. Sử dụng **shadcn/ui**.
2. **`shared/` (Molecules)**: Layout components (Navbar, Sidebar), Form wrappers.
3. **`features/` (Organisms)**: Domain-specific components. Ví dụ: `ProjectList`, `IssueTable`.

---

## 🎨 Design System: shadcn/ui

Chúng ta không build UI từ đầu mà dùng **shadcn/ui** (dựa trên Tailwind + Radix UI).
- **Style**: Modern, clean, accessible.
- **Customization**: Chỉnh sửa trực tiếp file code trong `components/ui`.

---

## 💡 Component Patterns

### 1. Presentational vs Container

**Container (Feature Component):**
- Handle logic, fetch data, quản lý state.
- Path: `components/features/projects/ProjectList.tsx`

**Presentational (UI Component):**
- Chỉ nhận props và hiển thị (Pure).
- Path: `components/features/projects/ProjectCard.tsx`

### 2. Composition Pattern

```tsx
<DashboardLayout>
  <ProjectHeader title="My Projects">
    <CreateProjectButton />
  </ProjectHeader>
  <ProjectList />
</DashboardLayout>
```

---

## 📝 Example: ProjectCard

```typescript
// apps/web/src/components/features/projects/ProjectCard.tsx
'use client';

import { Card } from '@/components/ui/card';
import { Badge } from '@/components/ui/badge';
import { Project } from '@/types';

interface ProjectCardProps {
  project: Project;
}

/**
 * ProjectCard Component
 * Display project summary info
 */
export function ProjectCard({ project }: ProjectCardProps) {
  return (
    <Card className="hover:border-primary p-4 transition-all">
      <h3 className="font-bold">{project.name}</h3>
      <p className="text-muted-foreground text-sm">{project.description}</p>
      <Badge className="mt-2">{project.status}</Badge>
    </Card>
  );
}
```

---

## ✅ Quality Standards

- [ ] **Accessibility**: Phải có labels cho người khiếm thị nếu dùng icon-only buttons.
- [ ] **Responsive**: Test trên Mobile (iPhone), Tablet (iPad), và Desktop.
- [ ] **Hydration**: Tránh mismatch giữa Server và Client render (không dùng `window` trực tiếp ở top-level).
- [ ] **JSDoc**: Tất cả function và component quan trọng phải có comment.

---

*Last Updated: 2026-02-11*
