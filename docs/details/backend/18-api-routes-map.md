# API Routes Map (WorkflowHub)

**Version:** v1.0.0
**Status:** 🗺️ API Blueprint
**Base URL:** `/api/v1`

Tài liệu này tổng hợp toàn bộ lộ trình API của hệ thống, giúp định hình rõ ràng các tính năng sẽ được xây dựng.

---

## 🔐 Auth & User Profile Module
*Quản lý danh tính và thông tin cá nhân.*

| Method | Endpoint | Description | Auth |
| :--- | :--- | :--- | :--- |
| `POST` | `/auth/register` | Đăng ký tài khoản mới | Public |
| `POST` | `/auth/login` | Đăng nhập & Nhận JWT | Public |
| `POST` | `/auth/logout` | Đăng xuất (Vô hiệu hóa Refresh Token) | Auth |
| `POST` | `/auth/refresh` | Làm mới Access Token | Public |
| `GET` | `/users/me` | Lấy thông tin cá nhân hiện tại | Auth |
| `PUT` | `/users/me` | Cập nhật Profile (Avatar, Display Name) | Auth |
| `PUT` | `/users/me/preferences` | Cập nhật cài đặt cá nhân (Theme, Language) | Auth |

---

## 🏢 Organization & Membership Module
*Trụ cột của Multi-tenant. Mọi Organization ID (`orgId`) đều bắt buộc trong URL.*

| Method | Endpoint | Description | Permission |
| :--- | :--- | :--- | :--- |
| `GET` | `/organizations` | Danh sách các tổ chức người dùng tham gia | Auth |
| `POST` | `/organizations` | Tạo tổ chức mới (Người tạo làm Owner) | Auth |
| `GET` | `/organizations/:orgId` | Chi tiết tổ chức & Cấu hình | Member |
| `PUT` | `/organizations/:orgId` | Cập nhật thông tin tổ chức | Admin/Owner |
| `GET` | `/organizations/:orgId/members` | Danh sách thành viên trong tổ chức | Member |
| `POST` | `/organizations/:orgId/members/invite`| Mời thành viên mới qua Email | Admin/Owner |
| `PATCH` | `/organizations/:orgId/members/:id`| Cập nhật Role cho thành viên | Admin/Owner |
| `DELETE`| `/organizations/:orgId/members/:id`| Xóa/Rời khỏi tổ chức | Member |

---

## 📁 Projects & Content Module
*Không gian làm việc cụ thể.*

| Method | Endpoint | Description | Permission |
| :--- | :--- | :--- | :--- |
| `GET` | `/:orgId/projects` | Danh sách dự án trong Org | Member |
| `POST` | `/:orgId/projects` | Tạo dự án mới | Admin/Owner |
| `GET` | `/:orgId/projects/:id` | Chi tiết một dự án | Member |
| `PUT` | `/:orgId/projects/:id` | Cập nhật dự án | Admin/Owner |
| `GET` | `/:orgId/projects/:id/stats` | Thống kê nhanh (Issue, Task count) | Member |

---

## 🎫 Issues & Tasks Module
*Quản lý công việc thực thi.*

| Method | Endpoint | Description | Permission |
| :--- | :--- | :--- | :--- |
| `GET` | `/:orgId/issues` | Danh sách Issues toàn Org | Member |
| `POST` | `/:orgId/issues` | Tạo Issue mới (Bug/Feature) | Member |
| `GET` | `/:orgId/issues/:id/tasks` | Danh sách task liên quan đến Issue | Member |
| `POST` | `/:orgId/tasks` | Tạo Task cụ thể | Member |
| `PATCH` | `/:orgId/tasks/:id/status`| Cập nhật trạng thái Task | Assignee/Admin |
| `POST` | `/:orgId/tasks/:id/assign`| Giao việc cho người khác | Admin |

---

## 🤖 AI & Automation Module (Phase 2 Focus)
*Trí tuệ nhân tạo và tri thức.*

| Method | Endpoint | Description | Permission |
| :--- | :--- | :--- | :--- |
| `GET` | `/:orgId/agents` | Danh sách AI Agents available | Member |
| `POST` | `/:orgId/agents` | Tạo/Cấu hình Agent mới | Admin/Owner |
| `POST` | `/:orgId/chats` | Khởi tạo phiên chat mới | Member |
| `GET` | `/:orgId/chats/:id/messages`| Lấy lịch sử tin nhắn | Member |
| `POST` | `/:orgId/chats/:id/send` | Gửi tin nhắn và nhận phản hồi AI | Member |
| `POST` | `/:orgId/documents/ingest` | Tải tài liệu lên để AI học (RAG) | Admin/Owner |

---

## 🏷️ Master Data & Settings Module
*Danh mục và cài đặt hệ thống.*

| Method | Endpoint | Description | Permission |
| :--- | :--- | :--- | :--- |
| `GET` | `/:orgId/lookups/categories` | Lấy danh sách Categories | Member |
| `POST` | `/:orgId/lookups/categories` | Tạo Category mới | Admin |
| `GET` | `/:orgId/lookups/tags` | Lấy danh sách Tags | Member |
| `GET` | `/:orgId/lookups/roles` | Lấy danh sách Role định nghĩa | Admin |

---

## 💡 Quy tắc chung (API Conventions)
1. **Response Format:** Mọi API đều trả về dạng: `{ success: boolean, data: any, message?: string }`.
2. **Error Codes:** Sử dụng chuẩn HTTP Status Codes (401: Unauthorized, 403: Forbidden, 404: Not Found, 422: Unprocessable Entity).
3. **Multi-tenant Key:** `orgId` là bắt buộc cho 90% các API để đảm bảo cô lập dữ liệu.

---

*Last Updated: 2026-02-11*
