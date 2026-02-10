# Frontend Coding Standards & Styling Rules

**Version:** v1.0.0
**Status:** 🛡️ Enforcement Rules
**Skills:** `frontend-developer`, `nextjs-best-practices`

Tài liệu này quy định các tiêu chuẩn bắt buộc về mã| [07-ai-integration.md](07-ai-integration.md) | AI Chat & Workflow UI | Phase 2 |
| [08-frontend-standards.md](08-frontend-standards.md) | **NEW:** Coding, Style, SCSS & CSS Var Standards | P0 |
| [09-ui-ux-patterns.md](09-ui-ux-patterns.md) | **NEW:** Responsive, Config-driven & l10n UX Patterns | P0 |

---

## 💻 Code Standards

### 1. ESLint & JSDoc
- **ESLint:** Bắt buộc tuân thủ bộ rule đã cấu hình (no unused vars, explicit returns, v.v.).
- **JSDoc:** Mọi Component, Hook, và Service function đều phải có JSDoc mô tả mục đích, tham số (`@param`) và giá trị trả về (`@returns`).

### 2. Props Destructuring
- Nếu một Component có **nhiều hơn 2 props**, bắt buộc phải xuống dòng khi destructuring để dễ đọc.
```tsx
// ✅ Good
function UserProfile({
  name,
  email,
  avatar,
  role,
  status
}: UserProfileProps) { ... }

// ❌ Bad
function UserProfile({ name, email, avatar, role, status }: UserProfileProps) { ... }
```

### 3. Centralized Constants (Single Source of Truth)
- Tất cả các giá trị fix cứng (API Endpoints, App Routes, Magic Numbers, Status Labels) phải được tập trung tại `src/constants/index.ts`.
- Tuyệt đối không hard-code chuỗi ký tự trực tiếp trong components.

### 4. Data-driven Navigation (SoC)
- **Không hard-code** danh sách menu trong Header, Sidebar hay Footer.
- Toàn bộ cấu trúc navigation phải được định nghĩa trong `src/constants/navigation.ts`.
- Components chỉ đóng vai trò "Render" dựa trên cấu trúc dữ liệu được cung cấp. Việc này giúp dễ dàng thay đổi menu hoặc phân quyền (Permissons) mà không cần can thiệp vào UI logic.

### 5. Page-based CRUD (Anti-Modal Strategy)
- **Tuyệt đối không dùng Modal** cho các thao tác CRUD chính (Tạo mới, Chỉnh sửa).
- Lý do: Modal phá vỡ focus, gây khó khăn cho việc scroll dữ liệu dài và cực kỳ tệ trên Mobile.
- **Giải pháp:** Sử dụng các trang riêng biệt hoặc cơ chế "Detail View" nguyên trang. Mọi hành động CRUD phải có URL riêng (ví dụ: `/projects/new`, `/projects/:id/edit`) để người dùng có thể dùng nút Back của trình duyệt hoặc chia sẻ link.

### 6. Logic Encapsulation in Hooks (SoC)
- **UI và Logic không trộn lẫn:** Logic nghiệp vụ, state management và side effects phải được đóng gói 100% trong Custom Hooks.
- **Hook không render UI:** Hook chỉ trả về dữ liệu và các hàm điều khiển (`data`, `methods`).
- **Độc lập với DOM/Layout:** Logic trong hook không được phụ thuộc vào cấu trúc DOM hay Layout cụ thể. UI có thể thay đổi hoàn toàn nhưng logic vẫn phải hoạt động đúng.
- **Non-blocking UX:** Khi thực hiện các tác vụ tốn thời gian (Create/Update), UI không nên "treo" ở trang hiện tại. Hãy chuyển hướng người dùng về trang đích ngay lập tức và quản lý tiến trình (`progress`) thông qua hệ thống **Toast thông báo**.

### 7. Localization-ready (l10n-ready)
- **Tuyệt đối không dùng Literal Strings:** Không viết text tiếng Việt hay tiếng Anh trực tiếp trong code (ví dụ: `<h1>Tạo dự án</h1>` ❌).
- **Translation Keys:** Sử dụng hệ thống translation keys (ví dụ: `project.create_title`, `common.save` ✅).
- **Data-driven Text:** Toàn bộ text hiển thị phải thông qua hàm dịch (ví dụ: `t('post.create')` thay vì `"Create Post"`). Việc này giúp hệ thống sẵn sàng mở rộng đa ngôn ngữ (i18n) bất cứ lúc nào mà không cần sửa code UI.

### 8. State Slicing & SoC (Single Responsibility Principle)
- **Tránh God Object:** Tuyệt đối không để toàn bộ state của ứng dụng vào một store duy nhất.
- **Slicing Pattern:** Chia nhỏ state thành các store riêng biệt dựa trên "Concern". Một store chỉ chịu trách nhiệm cho một mảng nghiệp vụ duy nhất.
  - Ví dụ: `useThemeStore`, `useConfigStore` (language, preferences), `useLayoutStore` (sidebar, overlays), `useAuthStore`.
- **Độc lập:** Các store nên hoạt động độc lập, tránh phụ thuộc chéo (circular dependency) giữa các store.

### 1. REM-based Sizing (No Pixel Policy)
- **Tuyệt đối không dùng `px`** cho các thuộc tính kích thước (width, height, padding, margin, font-size).
- **Tiêu chuẩn:** Sử dụng `rem` làm đơn vị đo lường cơ bản để đảm bảo tính khả dụng (Accessibility) và co giãn chuẩn xác theo thiết lập của trình duyệt.

### 2. CSS Variables & Design Tokens (No Hard-coded Values)
- **Không tự sinh giá trị:** Cấm viết trực tiếp các giá trị màu sắc (`#ffffff`), đổ bóng (`box-shadow`), độ nhòe (`blur`), hay dải màu (`linear-gradient`) trong code styling.
- **Khai báo tập trung:** Toàn bộ Design Tokens phải được khai báo trong `:root` (thường là trong file `globals.scss`).
- **Sử dụng `var()`:** Mọi thuộc tính CSS phải truy xuất giá trị thông qua biến CSS (ví dụ: `color: var(--primary-color)`).

### 3. SCSS Implementation (Modular Styling)
- **Sử dụng SCSS thay vì CSS:** Để tận dụng sức mạnh của Nesting, Mixins và Variables, đồng thời hỗ trợ tốt cho việc ghi đè (override) style của **shadcn/ui**.
- **Global Styles:** Toàn bộ cấu trúc Design System được định nghĩa trong các file `.scss` (ví dụ: `src/styles/variables.scss`, `src/styles/mixins.scss`).

---

## 🎨 Layout Philosophy

### 1. Fluid & Content-Driven Layout
- **Fluid Layout:** Layout phải co giãn theo Viewport. Tránh fix cứng `width` và `height`.
- **Content-Driven:** Kích thước của element nên phụ thuộc vào nội dung bên trong nó hơn là các con số cố định.
- **Tránh số đo cố định:** Sử dụng `rem`, `%`, `vh`, `vw`, `flex-1`, `grid-auto-fit` để đạt được sự linh hoạt.

### 2. Resilient UI (Giao diện kiên cường)
- **Long Text handling:** Sử dụng `truncate`, `line-clamp` hoặc cơ chế co giãn để UI không bị vỡ khi gặp text quá dài.
- **Data Load:** UI phải trông vẫn đẹp ngay cả khi có rất ít hoặc rất nhiều dữ liệu.
- **Small Screens:** Đảm bảo tính khả dụng (Usability) trên màn hình nhỏ mà không bị mất thông tin quan trọng.

### 3. Layout Tools Preference
- Ưu tiên sử dụng **Flexbox** và **CSS Grid** cho mọi bố cục.
- Sử dụng `max-width` cho containers để đảm bảo không quá rộng trên màn hình lớn.
- Sử dụng `min-height` thay vì `height` cố định để cho phép element giãn nở theo nội dung.

---

## ✅ Checklist cho Developer
- [ ] Đã chạy ESLint trước khi commit.
- [ ] Đã viết JSDoc cho logic mới.
- [ ] Các hằng số mới đã được đưa vào `constants.js`.
- [ ] UI đã được test với text siêu dài và màn hình nhỏ nhất.
- [ ] Không có giá trị `width` hay `height` fix cứng không cần thiết.

---

*Last Updated: 2026-02-11*
