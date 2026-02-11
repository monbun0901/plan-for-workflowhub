# Auth Layout

**Version:** v1.0.0
**Status:** 🔐 Auth Pages
**Path:** `src/app/(auth)/layout.tsx`

---

## 🏗️ Architecture

Auth Layout là layout riêng biệt cho các trang **không cần Sidebar hay Header** (Login, Register, Forgot Password).

### Layout Flow
```
┌──────────────────────────────────┐
│                                  │
│     ┌────────────────────┐       │
│     │   Brand / Logo     │       │
│     │                    │       │
│     │   Auth Form Card   │       │
│     │                    │       │
│     │   Footer Links     │       │
│     └────────────────────┘       │
│                                  │
└──────────────────────────────────┘
         (Centered, max-w-md)
```

---

## 🏗️ Boilerplate Code

```tsx
/**
 * @follows senior-architect: Isolated Auth Layout
 * Không kế thừa Sidebar/Header từ Dashboard
 */
export default function AuthLayout({ children }: { children: React.ReactNode }) {
  return (
    <div className="flex min-h-screen items-center justify-center bg-background p-4">
      <div className="w-full max-w-md space-y-6">
        {/* Brand */}
        <div className="text-center">
          <Logo className="mx-auto h-10 w-auto" />
          <h1 className="mt-4 text-2xl font-bold">{t('auth.welcome')}</h1>
        </div>

        {/* Auth Form (Login/Register/Reset) */}
        <div className="rounded-xl border bg-card p-6 shadow-sm">
          {children}
        </div>

        {/* Footer */}
        <p className="text-center text-sm text-muted-foreground">
          {t('auth.footer_text')}
        </p>
      </div>
    </div>
  );
}
```

---

## 📐 Design Rules

- **Centered Layout:** Card nằm giữa màn hình trên mọi thiết bị.
- **`max-w-md`:** Giới hạn chiều rộng form để giữ trải nghiệm đọc tốt.
- **No Sidebar / No Header:** Layout hoàn toàn cô lập.
- **l10n-ready:** Mọi text thông qua `t()`.

---

## 📚 Related
- [01-shell-layout.md](01-shell-layout.md) — Dashboard layout (so sánh)
- [../patterns/01-localization.md](../patterns/01-localization.md) — l10n pattern

---

*Last Updated: 2026-02-11*
