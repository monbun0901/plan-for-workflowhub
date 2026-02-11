# Config Store

**Version:** v1.0.0
**Status:** ⚙️ App Preferences State
**Path:** `src/stores/config.store.ts`

---

## 🎯 Responsibility

Quản lý **duy nhất** các cài đặt ứng dụng mà người dùng có thể tùy chỉnh: theme, language, preferences.

---

## 📝 Store Spec

```typescript
// src/stores/config.store.ts
import { create } from 'zustand';
import { persist } from 'zustand/middleware';

type Theme = 'light' | 'dark' | 'system';
type Language = 'vi' | 'en';

interface ConfigState {
  theme: Theme;
  language: Language;
  setTheme: (theme: Theme) => void;
  setLanguage: (lang: Language) => void;
}

export const useConfigStore = create<ConfigState>()(
  persist(
    (set) => ({
      theme: 'dark',
      language: 'vi',
      setTheme: (theme) => set({ theme }),
      setLanguage: (language) => set({ language }),
    }),
    { name: 'app-config' }
  )
);
```

---

## 🔑 Rules

- **Persist:** Dùng `persist` middleware để giữ preferences qua sessions.
- **SRP:** Chỉ chứa app-level config. Không chứa sidebar state (→ `LayoutStore`) hay user info (→ `AuthStore`).
- **Type-safe:** Enum types cho `Theme` và `Language` để tránh magic strings.

---

## 📚 Related
- [../patterns/01-localization.md](../patterns/01-localization.md) — l10n sử dụng `language` từ store này
- [01-auth-store.md](01-auth-store.md) — So sánh: auth vs config concern

---

*Last Updated: 2026-02-11*
