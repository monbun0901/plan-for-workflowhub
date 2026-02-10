# UI/UX Responsive Patterns & Shell Architecture

**Version:** v1.0.0
**Status:** 🎨 Design System & Layout Engine
**Skills:** `frontend-developer`, `senior-architect`

Tài liệu này định nghĩa cấu trúc Layout "Ready-to-use" cho WorkflowHub, giúp developer không phải suy nghĩ trừu tượng khi triển khai thực tế.

---

## 🏛️ The Main Shell Architecture

Cấu trúc Shell của chúng ta phải đạt tới cấp độ **"Adaptive"** (Thích nghi), không chỉ là Responsive đơn thuần.

### 1. Header Linh động (Contextual Header)
Header không chỉ cố định các menu, mà phải thay đổi theo ngữ cảnh của ứng dụng:
- **Default:** Hiện Logo, Search, User Nav.
- **Inside Project:** Hiện Tên dự án, Breadcrumbs, và các nút thao tác nhanh (Quick Actions).
- **Mobile Mode:** Thu gọn vào nút Menu và hiện tiêu đề trang hiện tại.

### 2. Sidebar đa trạng thái (Triple-State Sidebar)
Hệ thống Sidebar được thiết kế để phục vụ 3 kịch bản sử dụng:

-   **🟢 Expand (Desktop > 1024px):** Chiều rộng **240px**, hiển thị đầy đủ Icon + Text label. Đây là trạng thái mặc định cho không gian làm việc rộng.
-   **🟡 Collapse (Desktop > 1024px):** Chiều rộng **64px**, chỉ hiển thị Icon. Người dùng có thể chủ động chuyển đổi để tăng diện tích hiển thị cho vùng Content.
-   **🔴 Mobile Overlay (Mobile < 1024px):** Sidebar ẩn hoàn toàn. Khi kích hoạt, nó sẽ xuất hiện dưới dạng **Fullscreen Overlay** (Drawer) che phủ toàn bộ màn hình để tối ưu trải nghiệm chạm.

---

## 🏗️ The "Golden Layout" Boilerplate (Adaptive Version)

```tsx
/**
 * @follows senior-architect: Adaptive Shell Pattern
 * Quản lý Sidebar và Header theo trạng thái linh động
 */
export default function DashboardLayout({ children }) {
  // Trạng thái đồng bộ qua Zustand: { isOpen, isCollapsed, toggleSidebar, toggleMobile }
  const { isCollapsed, isOpen, toggleSidebar, setOpen } = useSidebarStore();

  return (
    <div className="relative flex min-h-screen bg-background text-foreground">
      {/* 1. MOBILE OVERLAY (Drawer Fullscreen) */}
      <div 
        className={cn(
          "fixed inset-0 z-[100] bg-background/80 backdrop-blur-sm lg:hidden",
          isOpen ? "block" : "hidden"
        )}
        onClick={() => setOpen(false)}
      >
        <aside 
          className="h-full w-[85%] max-w-[300px] border-r bg-card p-6 shadow-xl"
          onClick={(e) => e.stopPropagation()}
        >
          {/* Mobile Sidebar Content */}
          <SidebarContent fullWidth />
        </aside>
      </div>

      {/* 2. DESKTOP SIDEBAR (Expand/Collapse) */}
      <aside 
        className={cn(
          "fixed left-0 top-0 z-50 h-screen border-r bg-card transition-all duration-300 ease-in-out hidden lg:block",
          isCollapsed ? "w-[64px]" : "w-[240px]"
        )}
      >
        <SidebarContent collapsed={isCollapsed} />
      </aside>

      {/* 3. MAIN WRAPPER */}
      <div 
        className={cn(
          "flex flex-1 flex-col transition-all duration-300",
          "lg:ml-[240px]", 
          isCollapsed && "lg:ml-[64px]"
        )}
      >
        <header className="sticky top-0 z-40 h-14 w-full border-b bg-background/80 backdrop-blur-md">
          <div className="flex h-full items-center justify-between px-4">
            <div className="flex items-center gap-4">
              {/* Desktop Toggle Button */}
              <Button variant="ghost" size="icon" onClick={toggleSidebar} className="hidden lg:flex">
                <MenuIcon className="h-4 w-4" />
              </Button>
              
              {/* Mobile Menu Button */}
              <Button variant="ghost" size="icon" onClick={() => setOpen(true)} className="lg:hidden">
                <MenuIcon className="h-4 w-4" />
              </Button>
              
              <Breadcrumbs />
            </div>
            <UserNav />
          </div>
        </header>

        <main className="flex-1 p-4 md:p-6 lg:p-8">
          <div className="mx-auto max-w-screen-2xl">
            {children}
          </div>
        </main>
      </div>
    </div>
  );
}
```

---

## 🧬 Fluid Component Patterns

### 1. The Resilient Grid (Auto-fill)
Dùng cho danh sách Projects hoặc Task Cards.
```tsx
// Không dùng grid-cols-3 vì sẽ vỡ ở màn nhỏ. Dùng auto-fill.
<div className="grid grid-cols-1 gap-4 sm:grid-cols-2 lg:grid-cols-3 xl:grid-cols-4">
  {items.map(item => <Card key={item.id} />)}
</div>
```

### 2. The Content-Centric Container
Dùng cho trang xem chi tiết tài liệu/issue.
```tsx
<article className="mx-auto max-w-3xl space-y-6 px-4">
  {/* Content ở đây luôn nằm giữa và không quá rộng trên màn 4K */}
  <h1>{title}</h1>
  <div className="prose dark:prose-invert">
    {content}
  </div>
</article>
```

---

## 🛠️ Data-driven Navigation Pattern (SoC)

Để tuân thủ nguyên tắc Separation of Concerns (SoC), chúng ta tách biệt hoàn toàn phần dữ liệu (Menu items) khỏi phần hiển thị.

### 1. Cấu hình Navigation (Single Source of Truth)
```typescript
// src/constants/navigation.ts
export const DASHBOARD_NAV = [
  {
    title: "Overview",
    href: "/dashboard",
    icon: LayoutIcon,
    roles: ["owner", "admin", "member"]
  },
  {
    title: "Projects",
    href: "/projects",
    icon: FolderIcon,
  },
  // Dễ dàng thêm mới hoặc đổi vị trí mà không cần sửa UI code
];
```

### 2. Render Component (Generic)
```tsx
// src/components/layout/SidebarNav.tsx
export function SidebarNav({ items, isCollapsed }) {
  return (
    <nav className="space-y-2">
      {items.map((item) => (
        <NavItem 
          key={item.href}
          icon={item.icon}
          label={item.title}
          href={item.href}
          isCollapsed={isCollapsed}
        />
      ))}
    </nav>
  );
}
```

---

## � Page-based CRUD Pattern

Chúng ta loại bỏ hoàn toàn Modal cho các tác vụ CRUD để tối ưu hóa khả năng tập trung (Focus) và trải nghiệm di động.

### 1. Cấu trúc URL chuẩn
Mọi thao tác dữ liệu đều gắn liền với một Route cụ thể:
- `GET /projects`: Danh sách dự án.
- `GET /projects/new`: Trang tạo mới dự án (Không phải Modal).
- `GET /projects/:id`: Trang chi tiết.
- `GET /projects/:id/edit`: Trang chỉnh sửa.

### 2. Ưu điểm của Page-based
- **Focus:** Người dùng tập trung hoàn toàn vào form mà không bị xao nhãng bởi nội dung nền.
- **Deep Linking:** Có thể gửi link trực tiếp cho một trang đang edit.
- **Form Persistence:** Dễ dàng lưu trạng thái form vào URL hoặc local storage mà không sợ mất khi đóng Modal nhầm.
- **Native Scroll:** Tận dụng thanh cuộn tự nhiên của trình duyệt, cực kỳ mượt mà trên Mobile.

### 3. Layout cho CRUD Page
```tsx
// src/app/(dashboard)/projects/new/page.tsx
export default function NewProjectPage() {
  return (
    <div className="mx-auto max-w-2xl space-y-8">
      <header className="flex items-center justify-between">
        <div>
          <h1 className="text-2xl font-bold">Create New Project</h1>
          <p className="text-muted-foreground">Bắt đầu một hành trình mới.</p>
        </div>
        <Button variant="ghost" onClick={() => router.back()}>Cancel</Button>
      </header>

      <div className="rounded-lg border bg-card p-6">
        <ProjectForm />
      </div>
    </div>
  );
}
```

---

## ⚡ Non-blocking Async Progress Pattern

Để đảm bảo trải nghiệm mượt mà, các tác vụ nặng (như tạo Project, import tài liệu) không được chặn (block) người dùng tại trang hiện tại.

### 1. Luồng xử lý mẫu (Toast-driven)
1. Người dùng nhấn **Submit** ở trang `/projects/new`.
2. Hook kích hoạt API call.
3. Ứng dụng **chuyển hướng ngay lập tức** về `/projects`.
4. Một **Toast thông báo** hiện lên thông báo: *"Đang tạo dự án..."* (với loading spinner).
5. Khi hoàn tất, Toast tự động cập nhật trạng thái Success hoặc Error.

### 2. Implementation Example
```tsx
// src/hooks/useProjectActions.ts
export function useCreateProject() {
  const router = useRouter();
  const { toast } = useToast();

  return useMutation({
    mutationFn: (data) => projectsService.create(data),
    onMutate: () => {
      // Chuyển hướng ngay lập tức
      router.push('/projects');
      toast.loading("Đang khởi tạo dự án của bạn...");
    },
    onSuccess: () => {
      toast.success("Dự án đã được tạo thành công!");
    },
    onError: (error) => {
      toast.error(`Lỗi: ${error.message}`);
    }
  });
}
```

---

## 🌍 Localization-ready (l10n) Pattern

Hệ thống được thiết kế để tách biệt hoàn toàn nội dung (Content) khỏi mã nguồn (Code).

### 1. Dictionary Structure (Conceptual)
```json
// public/locales/vi/project.json
{
  "create_title": "Tạo dự án mới",
  "name_label": "Tên dự án",
  "placeholder": "Nhập tên dự án..."
}
```

### 2. Usage in Components
```tsx
// @follows senior-architect: l10n-ready
export function ProjectHeader() {
  const { t } = useTranslation();

  return (
    <header>
      <h1>{t('project.create_title')}</h1>
      <Button>{t('common.actions.save')}</Button>
    </header>
  );
}
```

### 3. Tại sao chọn cách này?
- **Global Compliance:** Dễ dàng thêm tiếng Anh, tiếng Nhật... chỉ bằng cách thêm file JSON.
- **Consistent Terminology:** Đảm bảo một thuật ngữ (ví dụ: "Sửa") được dùng thống nhất toàn app thông qua một key duy nhất.
- **No Translation Bugs:** Tránh việc sót text chưa dịch khi code giao diện.

---

## 🛡️ Resilient UI Checklist
- [ ] **Sidebar Overlay:** Khi ở Mobile, Sidebar mở ra phải có lớp phủ (Backdrop).
- [ ] **Empty States:** Luôn có UI cho trường hợp 0 item (Illustrations + Nút tạo mới).
- [ ] **Loading Skeletons:** Dùng `loading.tsx` để giữ layout ổn định khi chờ data.
- [ ] **Text Overflow:** Sử dụng `break-words` cho mô tả và `truncate` cho tiêu đề dài.

---

*Last Updated: 2026-02-11*
