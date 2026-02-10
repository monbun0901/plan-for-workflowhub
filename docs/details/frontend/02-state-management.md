# State Management Strategy

**Version:** v1  
**Tools:** Zustand, TanStack Query

---

## 🏗️ State Philosophy (SoC & Slicing)

Hệ thống tuân thủ nghiêm ngặt nguyên tắc **Separation of Concerns (SoC)** và **Single Responsibility Principle (SRP)**. Chúng ta tuyệt đối tránh việc tạo ra một "God Object" (một store chứa tất cả mọi thứ).

Thay vào đó, ứng dụng được chia làm 2 tầng state:
1. **Server State (TanStack Query)**: Quản lý toàn bộ dữ liệu từ API.
2. **Global UI State (Zustand Slices)**: Được chia nhỏ thành các store độc lập.

---

## 🧩 Store Slicing Pattern

Mỗi store chỉ chịu trách nhiệm cho một "Concern" duy nhất. Việc chia nhỏ này giúp code dễ bảo trì hơn và tránh việc re-render không cần thiết trên toàn bộ ứng dụng.

### 1. Layout Store (UI Control)
Chỉ quản lý các trạng thái hiển thị của giao diện.
```typescript
// src/stores/layout.store.ts
export const useLayoutStore = create<LayoutState>((set) => ({
  isSidebarCollapsed: false,
  isMobileSearchOpen: false,
  toggleSidebar: () => set((state) => ({ isSidebarCollapsed: !state.isSidebarCollapsed })),
}));
```

### 2. Config Store (App Preferences)
Quản lý các cài đặt như ngôn ngữ, theme.
```typescript
// src/stores/config.store.ts
export const useConfigStore = create<ConfigState>()(
  persist(
    (set) => ({
      language: 'vi',
      theme: 'dark',
      setLanguage: (lang) => set({ language: lang }),
      setTheme: (theme) => set({ theme }),
    }),
    { name: 'app-config' }
  )
);
```

### 3. Auth Store (Identity)
Chỉ quản lý thông tin xác thực và hồ sơ người dùng.
```typescript
// src/stores/auth.store.ts
export const useAuthStore = create<AuthState>((set) => ({
  user: null,
  setAuth: (user) => set({ user }),
  clearAuth: () => set({ user: null }),
}));
```

---

## 🚀 Server State with TanStack Query

Chúng ta sử dụng TanStack Query cho tất cả các thao tác dữ liệu với API.

### Cấu hình QueryClient (Root Layout)
```tsx
const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      staleTime: 1000 * 60 * 5, // 5 minutes
      retry: 1,
    },
  },
});
```

### Tại sao dùng TanStack Query?
- ✅ **Automatic Caching**: Không cần gọi lại API nếu dữ liệu còn mới.
- ✅ **Pagination & Infinite Scroll**: Hỗ trợ sẵn.
- ✅ **Mutations**: Dễ dàng update UI sau khi CREATE/UPDATE/DELETE.
- ✅ **Request Deduplication**: Gộp nhiều request giống nhau thành 1.

---

## 💡 Best Practices

1. **Keep Stores Small**: Chia nhỏ stores theo domain thay vì 1 giant store.
2. **Client-side only Persist**: Cẩn thận khi dùng `persist` với SSR (Next.js).
3. **Selector Pattern**: Luôn dùng selectors để tránh re-render không cần thiết.
   ```typescript
   const user = useAuthStore(state => state.user); // ✅ Good
   const state = useAuthStore(); // ❌ Bad: re-render on any auth change
   ```

---

## 📚 Related Documents

- [03-api-service.md](03-api-service.md) - API client setup
- [05-hooks.md](05-hooks.md) - Using TanStack Query in hooks

---

*Last Updated: 2026-02-11*
