# Custom Hooks & Data Fetching

**Version:** v1  
**Skills:** `frontend-developer`, `nextjs-best-practices`

---

## 🏗️ The Hook Pattern

Chúng ta đóng gói toàn bộ logic gọi API và quản lý cache vào Custom Hooks sử dụng **TanStack Query**.

### Tại sao dùng Hooks?
- **Reuse logic**: Dùng lại query logic ở nhiều trang.
- **Clean Components**: Xóa sạch code gọi API khỏi view.
- **Testing**: Dễ dàng unit test cho business logic.

---

## 🟢 Example: useProjects Hook

```typescript
// apps/web/src/hooks/useProjects.ts
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query';
import { projectsService } from '@/services/projects.service';

export function useProjects(orgId: string) {
  const queryClient = useQueryClient();

  // 1. Fetch data
  const { data, isLoading } = useQuery({
    queryKey: ['projects', orgId],
    queryFn: () => projectsService.list(orgId),
    enabled: !!orgId,
  });

  // 2. Mutate data
  const createProject = useMutation({
    mutationFn: (newProject: any) => projectsService.create(orgId, newProject),
    onSuccess: () => {
      // Invalidate cache để load lại list mới nhất
      queryClient.invalidateQueries({ queryKey: ['projects', orgId] });
    },
  });

  return {
    projects: data?.data || [],
    isLoading,
    createProject: createProject.mutate,
    isCreating: createProject.isPending,
  };
}
```

---

## 🛠️ Common Hooks

| Hook | Purpose |
|------|---------|
| `useAuth` | Đăng nhập, đăng xuất, lấy user info |
| `useTenant` | Switch organization, lấy active org |
| `useDebounce` | Xử lý search input để tránh gọi API liên tục |
| `useLocalStorage` | Sync state với browser storage |

---

## 💡 Advanced: Optimistic Updates

Cho các hành động như Toggle Star, Complete Task, hãy update UI ngay lập tức trước khi server phản hồi:

```typescript
const completeTask = useMutation({
  onMutate: async (taskId) => {
    await queryClient.cancelQueries({ queryKey: ['tasks'] });
    const previous = queryClient.getQueryData(['tasks']);
    // Update local cache ngay lập tức
    queryClient.setQueryData(['tasks'], (old) => updateStatus(old, taskId));
    return { previous };
  },
  onError: (err, id, context) => {
    // Rollback nếu server lỗi
    queryClient.setQueryData(['tasks'], context.previous);
  }
});
```

---

## 🔀 Hooks vs Contexts — Phân biệt rõ ràng

Dự án sử dụng cả **Custom Hooks** (`hooks/`) và **React Contexts** (`app/contexts/`). Hai layer này có vai trò khác nhau:

| | Custom Hooks (`hooks/`) | Contexts (`app/contexts/`) |
|---|---|---|
| **Vị trí** | `src/hooks/` | `src/app/contexts/` |
| **Mục đích** | Reusable logic utilities, TanStack Query wrappers | Page-level logic orchestration cho từng feature |
| **Chứa gì** | `useQuery`, `useMutation`, `useDebounce`, `useLocalStorage` | `useEffect` fetch, `useMemo` filter/sort, event handlers, pagination |
| **Phạm vi** | Dùng lại ở nhiều nơi, không gắn với feature cụ thể | Gắn với 1 feature/page cụ thể (Tasks, Issues, ...) |
| **State source** | TanStack Query cache hoặc local state | Wrap Zustand stores, không thay thế chúng |
| **Ví dụ** | `useProjects()`, `useDebounce()`, `useAuth()` | `useTaskListContext()`, `useIssueFormContext()` |

### Khi nào dùng Hook?

- Logic **tái sử dụng** ở nhiều pages/components
- Đóng gói 1 API call đơn lẻ (query hoặc mutation)
- Utility logic không gắn với feature cụ thể (debounce, localStorage, etc.)

### Khi nào dùng Context?

- Orchestrate **nhiều stores** cùng lúc cho 1 page (fetch tasks + statuses + categories + members)
- Chứa **filtering, sorting, pagination** logic cho list page
- Chứa **event handlers** (handleEdit, handleDelete, handleView)
- Cung cấp **form dependencies** cho create/edit page (load statuses, members, categories...)

### Ví dụ minh họa

```tsx
// ❌ KHÔNG nên: Hook chứa logic đặc thù cho 1 page
function useTaskListPage() {
  // fetch 4 stores, filter, sort, paginate...
  // → Quá phức tạp, không tái sử dụng được
}

// ✅ NÊN: Context cho page-level logic
function TaskListProvider({ children }) {
  // fetch 4 stores, filter, sort, paginate, handlers
  return <TaskListContext.Provider value={...}>{children}</TaskListContext.Provider>
}

// ✅ NÊN: Hook cho logic tái sử dụng
function useDebounce(value, delay) { ... }
function useProjects(orgId) { ... }
```

---

## ✅ Best Practices Checklist

- [ ] Luôn đặt `queryKey` theo hierarchy (ví dụ: `['projects', orgId, projectId]`).
- [ ] Sử dụng `enabled: !!dependency` để tránh gọi API khi chưa có data cần thiết.
- [ ] Tách `mutations` ra khỏi `queries` cho rõ ràng.

---

## 📚 Related Documents

- [02-state-management.md](02-state-management.md) - Cache strategy
- [03-api-service.md](03-api-service.md) - Services used by hooks

---

*Last Updated: 2026-02-11*
