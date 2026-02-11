# Toast System & Non-blocking Async

**Version:** v1.0.0
**Status:** ⚡ UX Pattern
**Path:** `src/components/ui/toast/`, `src/hooks/useToast.ts`

---

## 🎯 Non-blocking UX Principle

Khi thực hiện các tác vụ nặng (Create/Update/Delete), UI **không block** người dùng ở trang hiện tại.

### Luồng xử lý (Toast-driven)
1. Người dùng nhấn **Submit** ở `/projects/new`.
2. Hook kích hoạt API call.
3. Ứng dụng **chuyển hướng ngay lập tức** về `/projects`.
4. Toast hiện: *"Đang tạo dự án..."* (loading spinner).
5. Khi hoàn tất → Toast cập nhật **Success** hoặc **Error**.

---

## 📝 Implementation Pattern

```tsx
// src/hooks/useProjectActions.ts
export function useCreateProject() {
  const router = useRouter();
  const { toast } = useToast();

  return useMutation({
    mutationFn: (data: CreateProjectDto) => projectsService.create(data),
    onMutate: () => {
      // Chuyển hướng ngay, không chờ
      router.push('/projects');
      toast.loading(t('project.creating'));
    },
    onSuccess: () => {
      toast.success(t('project.created_success'));
    },
    onError: (error) => {
      toast.error(t('common.error', { message: error.message }));
    }
  });
}
```

---

## 🎨 Toast Variants

| Variant | Use Case | Icon | Auto-dismiss |
|---------|----------|------|-------------|
| `loading` | Đang xử lý | Spinner | Không (chờ kết quả) |
| `success` | Thành công | ✅ Check | 4s |
| `error` | Thất bại | ❌ X | 8s (cần đọc lỗi) |
| `info` | Thông tin chung | ℹ️ Info | 4s |
| `warning` | Cảnh báo | ⚠️ Warning | 6s |

---

## 📐 Position & Styling

- **Vị trí:** Bottom-right (Desktop), Bottom-center (Mobile).
- **Stack:** Tối đa 3 toast cùng lúc, oldest bị đẩy ra.
- **Đơn vị:** Sử dụng `rem`, tokens từ `:root`.

---

## 📚 Related
- [../layouts/03-crud-page-layout.md](../layouts/03-crud-page-layout.md) — CRUD page submit flow
- [06-form-patterns.md](06-form-patterns.md) — Form submission triggers

---

*Last Updated: 2026-02-11*
