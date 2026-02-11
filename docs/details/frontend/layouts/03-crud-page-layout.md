# Page-based CRUD Layout

**Version:** v1.0.0
**Status:** 📑 Anti-Modal Strategy
**Path:** `src/app/(dashboard)/[feature]/new/page.tsx`, `src/app/(dashboard)/[feature]/[id]/edit/page.tsx`

---

## 🚫 Anti-Modal Policy

Tuyệt đối không dùng Modal cho các thao tác CRUD chính (Tạo mới, Chỉnh sửa). Modal phá vỡ focus, scroll tồi tệ trên Mobile, và không có URL riêng.

---

## 📋 Cấu trúc URL chuẩn

Mỗi thao tác CRUD gắn với một Route cụ thể:

| Action | Route | Layout |
|--------|-------|--------|
| List | `/projects` | Grid/Table view |
| Create | `/projects/new` | Form page (max-w-2xl) |
| Detail | `/projects/:id` | Detail view |
| Edit | `/projects/:id/edit` | Form page (max-w-2xl) |

---

## 🏗️ Boilerplate Code

```tsx
// src/app/(dashboard)/projects/new/page.tsx
export default function NewProjectPage() {
  const { t } = useTranslation();

  return (
    <div className="mx-auto max-w-2xl space-y-8">
      {/* Page Header */}
      <header className="flex items-center justify-between">
        <div>
          <h1 className="text-2xl font-bold">{t('project.create_title')}</h1>
          <p className="text-muted-foreground">{t('project.create_description')}</p>
        </div>
        <Button variant="ghost" onClick={() => router.back()}>
          {t('common.actions.cancel')}
        </Button>
      </header>

      {/* Form Container */}
      <div className="rounded-lg border bg-card p-6">
        <ProjectForm />
      </div>
    </div>
  );
}
```

---

## ✅ Ưu điểm của cách tiếp cận này

- **Focus:** Người dùng tập trung hoàn toàn vào form, không bị nội dung nền xao nhãng.
- **Deep Linking:** Có thể gửi link trực tiếp cho trang đang edit.
- **Form Persistence:** Dễ lưu trạng thái form vào URL/localStorage.
- **Native Scroll:** Thanh cuộn tự nhiên của trình duyệt, mượt mà trên Mobile.
- **Nút Back:** Hoạt động tự nhiên với browser history.

---

## 📐 Design Rules

- **`max-w-2xl`:** Giữ form ở chiều rộng đọc thoải mái.
- **`space-y-8`:** Khoảng cách đủ giữa header và form.
- **l10n-ready:** Mọi text thông qua `t()`.
- **Non-blocking Submit:** Sau khi submit, chuyển hướng ngay (xem `../components/04-toast-system.md`).

---

## 📚 Related
- [01-shell-layout.md](01-shell-layout.md) — Parent layout (DashboardLayout)
- [../components/04-toast-system.md](../components/04-toast-system.md) — Non-blocking async feedback
- [../components/06-form-patterns.md](../components/06-form-patterns.md) — Form validation UX

---

*Last Updated: 2026-02-11*
