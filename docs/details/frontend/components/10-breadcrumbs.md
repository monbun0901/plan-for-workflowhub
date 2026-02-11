# Breadcrumbs Component

**Version:** v1.0.0
**Status:** 🧩 Reusable Component
**Path:** `src/components/shared/Breadcrumbs.tsx`

---

## 🎯 Purpose

Navigation breadcrumbs for showing current page hierarchy.

---

## 📦 Props Interface

```typescript
interface BreadcrumbItem {
  label: string;
  href?: string;          // Undefined = current page (no link)
  icon?: ReactNode;
}

interface BreadcrumbsProps {
  items: BreadcrumbItem[];
  separator?: ReactNode;   // Default: ChevronRightIcon
}
```

---

## 🧬 Component Structure

```tsx
/**
 * @follows senior-architect: Navigation Hierarchy Pattern
 * Auto-generated hoặc manual breadcrumbs
 */
import { ChevronRightIcon } from '@heroicons/react/24/outline';
import Link from 'next/link';

export function Breadcrumbs({ items, separator }: BreadcrumbsProps) {
  const Separator = separator || <ChevronRightIcon className="h-4 w-4 text-gray-400" />;
  
  return (
    <nav aria-label="Breadcrumb" className="flex items-center gap-2 text-sm">
      {items.map((item, index) => {
        const isLast = index === items.length - 1;
        
        return (
          <div key={index} className="flex items-center gap-2">
            {item.href && !isLast ? (
              <Link
                href={item.href}
                className="flex items-center gap-1.5 text-gray-600 hover:text-gray-900"
              >
                {item.icon}
                <span>{item.label}</span>
              </Link>
            ) : (
              <span className="flex items-center gap-1.5 font-medium text-gray-900">
                {item.icon}
                <span>{item.label}</span>
              </span>
            )}
            
            {!isLast && Separator}
          </div>
        );
      })}
    </nav>
  );
}
```

---

## 📚 Usage Examples

### Basic Breadcrumbs
```tsx
<Breadcrumbs
  items={[
    { label: 'Projects', href: '/projects' },
    { label: 'Alpha Project', href: '/projects/123' },
    { label: 'Task #42' },  // Current page (no href)
  ]}
/>
```

### With Icons
```tsx
<Breadcrumbs
  items={[
    { label: 'Home', href: '/dashboard', icon: <HomeIcon /> },
    { label: 'Settings', href: '/settings', icon: <CogIcon /> },
    { label: 'Profile' },
  ]}
/>
```

### Custom Separator
```tsx
<Breadcrumbs
  items={breadcrumbItems}
  separator={<span className="text-gray-400">/</span>}
/>
```

---

## 🎨 Design Rules

- **Current Page:** Không có link và bold font-weight.
- **Responsive:** Truncate giữa items trên mobile (show first + last only).
- **A11y:** Use `aria-label="Breadcrumb"` và `aria-current="page"` cho item cuối.

---

*Last Updated: 2026-02-11*
