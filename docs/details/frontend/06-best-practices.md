# Frontend Best Practices

**Version:** v1  
**Skills:** `nextjs-best-practices`, `frontend-developer`

---

## ⚡ Performance Optimization

### 1. Image Optimization
Luôn sử dụng component `next/image` để tự động resize, lazy load và định dạng webp.
```tsx
<Image src={user.avatar} width={40} height={40} alt="User Avatar" />
```

### 2. Client-side Hydration
Tránh mismatch giữa Server và Client render bằng cách không dùng các giá trị browser (window.innerWidth) trực tiếp khi render lần đầu.

### 3. Caching với TanStack Query
- Set `staleTime` phù hợp để tránh refetch dữ liệu không cần thiết.
- Dùng `Prefetching` khi user hover vào link.

---

## ♿ Accessibility (a11y)

Hệ thống phải đạt tiêu chuẩn WCAG:
- **Semantic HTML**: Dùng đúng tags `<header>`, `<main>`, `<nav>`, `<footer>`.
- **Contrast**: Đảm bảo text tương phản tốt trên nền (sử dụng Tailwind colors như `text-muted-foreground`).
- **Keyboard Navigation**: User phải dùng được phím Tab để điều hướng.
- **ARIA Labels**: Bổ sung cho các components không có label (icon buttons).

---

## 📱 Responsive & Mobile-First

Sử dụng Tailwind breakpoints:
- `default`: Mobile
- `sm`: Tablet portrait
- `md`: Tablet landscape
- `lg`: Laptop
- `xl`: Desktop

**Rule:** Hãy design cho màn hình nhỏ nhất trước, sau đó dùng các prefix `md:`, `lg:` để mở rộng UI.

---

## 🛡️ Security

1. **XSS Prevention**: React tự động escape HTML, nhưng hãy cẩn thận với `dangerouslySetInnerHTML`.
2. **Environment Variables**: Không bao giờ đặt secret keys (private API keys) vào `NEXT_PUBLIC_`.
3. **Form Validation**: Luôn validate dữ liệu ở Client (Zod + React Hook Form) trước khi gửi lên Server.

---

## 🧪 Testing Strategy

1. **Unit Tests**: Kiểm tra các logic helper, utils.
2. **Component Tests**: Dùng React Testing Library cho các UI components lớn.
3. **E2E Tests**: (Phase 2) Dùng Playwright cho các luồng chính như Login, Create Project.

---

*Last Updated: 2026-02-11*
