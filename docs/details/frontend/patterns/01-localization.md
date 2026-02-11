# Localization-ready (l10n) Pattern

**Version:** v1.0.0
**Status:** 🌍 Cross-cutting Pattern
**Applies to:** Toàn bộ UI components

---

## 🎯 Principle

Tách biệt hoàn toàn nội dung hiển thị (Content) khỏi mã nguồn (Code). Mọi text phải thông qua translation keys.

---

## ❌ Cấm

```tsx
// ❌ Hard-coded text
<h1>Tạo dự án mới</h1>
<Button>Save</Button>
<p>Không có dữ liệu</p>
```

## ✅ Đúng

```tsx
// ✅ Translation keys
<h1>{t('project.create_title')}</h1>
<Button>{t('common.actions.save')}</Button>
<p>{t('common.empty_state')}</p>
```

---

## 📁 Dictionary Structure

```
public/locales/
├── vi/
│   ├── common.json        # Shared: actions, labels, errors
│   ├── auth.json           # Login, Register
│   ├── project.json        # Project-specific
│   ├── task.json
│   └── nav.json            # Navigation labels
└── en/
    ├── common.json
    ├── auth.json
    └── ...
```

### Ví dụ Dictionary
```json
// public/locales/vi/project.json
{
  "create_title": "Tạo dự án mới",
  "create_description": "Bắt đầu một hành trình mới",
  "name_label": "Tên dự án",
  "name_placeholder": "Nhập tên dự án...",
  "created_success": "Dự án đã được tạo thành công!",
  "empty_title": "Chưa có dự án nào",
  "empty_description": "Hãy tạo dự án đầu tiên của bạn"
}
```

---

## 🔑 Key Naming Convention

```
[domain].[context]_[element]

Ví dụ:
- project.create_title       → Tiêu đề trang tạo project
- common.actions.save        → Nút Save chung
- auth.login_button          → Nút đăng nhập
- nav.projects               → Menu label "Projects"
```

---

## 📚 Related
- [../stores/03-config-store.md](../stores/03-config-store.md) — `language` state
- [../../08-frontend-standards.md](../../08-frontend-standards.md) — Rule #7 (l10n-ready)

---

*Last Updated: 2026-02-11*
