# Resilient UI Patterns

**Version:** v1.0.0
**Status:** 🛡️ Cross-cutting Pattern
**Applies to:** Toàn bộ UI components

---

## 🎯 Principle

UI phải hoạt động tốt trong mọi tình huống: dữ liệu ít, dữ liệu nhiều, dữ liệu lỗi, hoặc không có dữ liệu.

---

## ✅ Resilient UI Checklist

- [ ] **Empty States:** Luôn có UI cho trường hợp 0 item (Illustrations + Nút tạo mới).
- [ ] **Loading Skeletons:** Dùng `loading.tsx` để giữ layout ổn định khi chờ data.
- [ ] **Error Boundaries:** `error.tsx` cho mỗi route segment.
- [ ] **Text Overflow:** Sử dụng `break-words` cho mô tả và `truncate` cho tiêu đề dài.
- [ ] **Long Lists:** Pagination hoặc virtual scroll cho danh sách > 100 items.
- [ ] **Sidebar Overlay:** Mobile sidebar phải có Backdrop overlay.
- [ ] **Image Fallback:** Luôn có `alt` text và fallback avatar/placeholder.
- [ ] **Form Validation:** Hiển thị lỗi inline ngay tại field, không dùng alert.

---

## 📝 Empty State Pattern

```tsx
interface EmptyStateProps {
  icon: LucideIcon;
  titleKey: string;
  descriptionKey: string;
  actionKey?: string;
  onAction?: () => void;
}

export function EmptyState({ icon: Icon, titleKey, descriptionKey, actionKey, onAction }: EmptyStateProps) {
  const { t } = useTranslation();

  return (
    <div className="flex flex-col items-center justify-center py-16 text-center">
      <Icon className="h-12 w-12 text-muted-foreground" />
      <h3 className="mt-4 text-lg font-semibold">{t(titleKey)}</h3>
      <p className="mt-2 text-sm text-muted-foreground">{t(descriptionKey)}</p>
      {actionKey && onAction && (
        <Button onClick={onAction} className="mt-6">
          {t(actionKey)}
        </Button>
      )}
    </div>
  );
}
```

---

## 📝 Loading Skeleton Pattern

```tsx
// src/app/(dashboard)/projects/loading.tsx
export default function ProjectsLoading() {
  return (
    <div className="grid grid-cols-1 gap-4 sm:grid-cols-2 lg:grid-cols-3">
      {Array.from({ length: 6 }).map((_, i) => (
        <Skeleton key={i} className="h-[10rem] rounded-lg" />
      ))}
    </div>
  );
}
```

---

## 🎨 Design Rules

- **Skeleton cùng layout:** Skeleton phải giữ nguyên layout structure của data view.
- **No "No data" text:** Sử dụng EmptyState component có illustration, không chỉ text.
- **Graceful Degradation:** UI không được vỡ khi thiếu dữ liệu.

---

## 📚 Related
- [../components/05-data-grid.md](../components/05-data-grid.md) — Grid pattern + Empty state
- [../../08-frontend-standards.md](../../08-frontend-standards.md) — Layout philosophy

---

*Last Updated: 2026-02-11*
