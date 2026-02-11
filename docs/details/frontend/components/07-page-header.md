# PageHeader Component

**Version:** v1.0.0
**Status:** 🧩 Reusable Component
**Path:** `src/components/shared/PageHeader.tsx`

---

## 🎯 Purpose

Unified page header pattern với title, description, và actions. Thay thế việc hard-code header ở mỗi page.

---

## 📦 Props Interface

```typescript
interface PageHeaderProps {
  title: string;                    // Page title
  description?: string;              // Optional subtitle/description
  actionSlot?: ReactNode;            // Right-side actions (buttons, filters...)
  backUrl?: string;                  // Optional back button
  breadcrumbs?: BreadcrumbItem[];    // Optional breadcrumb override
}
```

---

## 🧬 Component Structure

```tsx
/**
 * @follows senior-architect: Composition Pattern
 * Generic PageHeader với slots cho customization
 */
import { ArrowLeftIcon } from '@heroicons/react/24/outline';
import { Breadcrumbs } from './Breadcrumbs';

export function PageHeader({
  title,
  description,
  actionSlot,
  backUrl,
  breadcrumbs,
}: PageHeaderProps) {
  return (
    <header className="border-b border-gray-200 bg-white px-6 py-4">
      {breadcrumbs && <Breadcrumbs items={breadcrumbs} />}
      
      <div className="mt-2 flex items-center justify-between">
        <div className="flex items-center gap-3">
          {backUrl && (
            <Link href={backUrl} className="hover:opacity-70">
              <ArrowLeftIcon className="h-5 w-5" />
            </Link>
          )}
          <div>
            <h1 className="text-2xl font-bold text-gray-900">{title}</h1>
            {description && (
              <p className="mt-1 text-sm text-gray-500">{description}</p>
            )}
          </div>
        </div>
        
        {actionSlot && (
          <div className="flex items-center gap-2">{actionSlot}</div>
        )}
      </div>
    </header>
  );
}
```

---

## 📚 Usage Examples

### Simple Page Header
```tsx
<PageHeader 
  title="Projects" 
  description="Manage your projects"
/>
```

### With Action Buttons
```tsx
<PageHeader
  title="Tasks"
  actionSlot={
    <>
      <ViewToggle pageKey="tasks" />
      <Button href="/tasks/new">+ New Task</Button>
    </>
  }
/>
```

### With Back Button
```tsx
<PageHeader
  title="Edit Project"
  backUrl="/projects"
/>
```

### With Breadcrumbs
```tsx
<PageHeader
  title="Task Detail"
  breadcrumbs={[
    { label: 'Projects', href: '/projects' },
    { label: 'Alpha', href: '/projects/123' },
    { label: 'Task #42' },
  ]}
/>
```

---

## 🎨 Design Rules

- **Full Width:** Header luôn toàn bộ viewport width.
- **Sticky:** Optional sticky positioning với `sticky top-0 z-10`.
- **Responsive:** Action slot collapse vào dropdown trên mobile.

---

*Last Updated: 2026-02-11*
