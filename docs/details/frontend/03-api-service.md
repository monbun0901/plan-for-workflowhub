# API Service Layer

**Version:** v1  
**Tool:** Axios

---

## 🏗️ Centralized API Client

Chúng ta sử dụng một Axios instance duy nhất cho toàn bộ app để quản lý Headers và Interceptors tập trung.

### Axios Instance
```typescript
// apps/web/src/services/api.ts
import axios from 'axios';
import { useAuthStore } from '@/stores/auth.store';

export const api = axios.create({
  baseURL: process.env.NEXT_PUBLIC_API_URL,
  headers: { 'Content-Type': 'application/json' },
});
```

---

## 🔒 Interceptors (Auth Flow)

### 1. Request Interceptor
Tự động đính kèm `Authorization: Bearer <token>` vào mọi request.

```typescript
api.interceptors.request.use((config) => {
  const token = useAuthStore.getState().accessToken;
  if (token) config.headers.Authorization = `Bearer ${token}`;
  return config;
});
```

### 2. Response Interceptor (Token Refresh)
Tự động làm mới access token nếu nhận được lỗi 401.

```typescript
api.interceptors.response.use(
  (res) => res,
  async (error) => {
    if (error.response?.status === 401) {
      // Logic call refresh token API
      // Then retry original request
    }
    return Promise.reject(error);
  }
);
```

---

## 📦 Feature Services

Mỗi module (projects, tasks, issues) sẽ có service riêng.

### Example: Projects Service
```typescript
// apps/web/src/services/projects.service.ts
export const projectsService = {
  list: (orgId: string) => api.get(`/organizations/${orgId}/projects`),
  create: (orgId: string, data: any) => api.post(`/organizations/${orgId}/projects`, data),
  // ... other CRUD methods
};
```

---

## ✅ Best Practices

- **DTOs**: Định nghĩa interface cho Request/Response.
- **Error Handling**: Không handle lỗi chung ở service, hãy throw để UI/Hook xử lý.
- **Environment Variables**: Luôn dùng `NEXT_PUBLIC_` cho API URL.

---

## 📚 Related Documents

- [02-state-management.md](02-state-management.md) - Auth store reference
- [05-hooks.md](05-hooks.md) - How to use services in hooks

---

*Last Updated: 2026-02-11*
