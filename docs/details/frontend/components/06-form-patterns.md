# Form Patterns

**Version:** v1.0.0
**Status:** 🧩 Input Pattern
**Path:** `src/components/shared/forms/`, `src/hooks/useForm*.ts`

---

## 🎯 Form Philosophy

- **Logic trong Hook:** Toàn bộ validation, submit, error handling nằm trong Custom Hook.
- **UI chỉ Render:** Form component chỉ nhận `register`, `errors`, `handleSubmit` từ hook.
- **Page-based:** Form luôn ở trang riêng (`/new`, `/:id/edit`), không dùng Modal.

---

## 🏗️ Form Structure Pattern

```tsx
// src/app/(dashboard)/projects/new/page.tsx
export default function NewProjectPage() {
  // Logic 100% trong hook
  const { form, onSubmit, isSubmitting } = useCreateProjectForm();
  const { t } = useTranslation();

  return (
    <div className="mx-auto max-w-2xl space-y-8">
      <PageHeader
        titleKey="project.create_title"
        descriptionKey="project.create_description"
      />

      <form onSubmit={form.handleSubmit(onSubmit)} className="space-y-6">
        <FormField
          label={t('project.name_label')}
          error={form.formState.errors.name?.message}
        >
          <Input {...form.register('name')} placeholder={t('project.name_placeholder')} />
        </FormField>

        <FormField
          label={t('project.description_label')}
          error={form.formState.errors.description?.message}
        >
          <Textarea {...form.register('description')} />
        </FormField>

        <FormActions>
          <Button type="button" variant="ghost" onClick={() => router.back()}>
            {t('common.actions.cancel')}
          </Button>
          <Button type="submit" disabled={isSubmitting}>
            {t('common.actions.save')}
          </Button>
        </FormActions>
      </form>
    </div>
  );
}
```

---

## 📝 Form Hook Pattern

```tsx
// src/hooks/useCreateProjectForm.ts
export function useCreateProjectForm() {
  const form = useForm<CreateProjectDto>({
    resolver: zodResolver(createProjectSchema),
    defaultValues: { name: '', description: '' }
  });

  const createMutation = useCreateProject(); // TanStack Mutation + Toast

  const onSubmit = (data: CreateProjectDto) => {
    createMutation.mutate(data);
  };

  return { form, onSubmit, isSubmitting: createMutation.isPending };
}
```

---

## 🎨 Design Rules

- **`max-w-2xl`:** Giới hạn chiều rộng form.
- **`space-y-6`:** Khoảng cách giữa các field.
- **l10n-ready:** Mọi label, placeholder, error message qua `t()`.
- **Validation:** Schema-based với Zod.
- **Actions:** Cancel (ghost) bên trái, Submit (primary) bên phải.

---

## 📚 Related
- [../layouts/03-crud-page-layout.md](../layouts/03-crud-page-layout.md) — CRUD page layout
- [04-toast-system.md](04-toast-system.md) — Submit → Toast feedback

---

*Last Updated: 2026-02-11*
