# FormCard Component

**Version:** v1.0.0
**Status:** 🧩 Reusable Component
**Path:** `src/components/shared/FormCard.tsx`

---

## 🎯 Purpose

Card wrapper for forms in CRUD pages, providing consistent styling and layout.

---

## 📦 Props Interface

```typescript
interface FormCardProps {
  title?: string;                // Optional card title
  description?: string;          // Optional description
  children: ReactNode;           // Form fields
  actions?: ReactNode;           // Footer actions (Cancel/Submit buttons)
  maxWidth?: 'sm' | 'md' | 'lg' | 'xl' | '2xl';  // Default: 'xl'
}
```

---

## 🧬 Component Structure

```tsx
/**
 * @follows senior-architect: Card Container Pattern
 * Consistent form styling across create/edit pages
 */
export function FormCard({
  title,
  description,
  children,
  actions,
  maxWidth = 'xl',
}: FormCardProps) {
  return (
    <div className={cn('mx-auto w-full', `max-w-${maxWidth}`)}>
      <div className="rounded-lg border border-gray-200 bg-white shadow-sm">
        {/* Card Header */}
        {(title || description) && (
          <div className="border-b border-gray-200 px-6 py-4">
            {title && <h2 className="text-lg font-semibold">{title}</h2>}
            {description && (
              <p className="mt-1 text-sm text-gray-600">{description}</p>
            )}
          </div>
        )}
        
        {/* Card Body */}
        <div className="px-6 py-6">
          {children}
        </div>
        
        {/* Card Footer */}
        {actions && (
          <div className="border-t border-gray-200 bg-gray-50 px-6 py-4">
            <div className="flex justify-end gap-3">
              {actions}
            </div>
          </div>
        )}
      </div>
    </div>
  );
}
```

---

## 📚 Usage Examples

### Simple Form Card
```tsx
<FormCard
  title="Create New Project"
  description="Fill in the details below"
  actions={
    <>
      <Button variant="ghost" href="/projects">Cancel</Button>
      <Button type="submit">Create Project</Button>
    </>
  }
>
  <Input label="Project Name" {...register('name')} />
  <Textarea label="Description" {...register('description')} />
</FormCard>
```

### Without Header
```tsx
<FormCard
  maxWidth="md"
  actions={<Button type="submit">Save</Button>}
>
  <Input label="Email" type="email" />
  <Input label="Password" type="password" />
</FormCard>
```

### Multi-Section Form
```tsx
<FormCard
  title="Project Settings"
  actions={<Button type="submit">Save Changes</Button>}
>
  <section className="space-y-4">
    <h3 className="text-sm font-semibold text-gray-900">General</h3>
    <Input label="Name" />
    <Input label="Description" />
  </section>
  
  <hr className="my-6" />
  
  <section className="space-y-4">
    <h3 className="text-sm font-semibold text-gray-900">Advanced</h3>
    <Switch label="Enable notifications" />
  </section>
</FormCard>
```

---

## 🎨 Design Rules

- **Centered:** Always centered với `mx-auto` và max-width.
- **Spacing:** Consistent padding: `px-6 py-4` (header/footer), `px-6 py-6` (body).
- **Shadow:** Subtle shadow (`shadow-sm`) for depth.
- **Actions Alignment:** Luôn `justify-end` (Cancel trái, Submit phải).

---

*Last Updated: 2026-02-11*
