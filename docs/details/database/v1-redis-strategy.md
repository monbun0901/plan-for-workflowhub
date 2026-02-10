# Redis Caching Strategy

**Type:** In-Memory Data Store  
**Usage:** Caching, Session Management, Rate Limiting, AI Context Buffer

---

## 📋 Overview
Redis được sử dụng để giảm tải cho MySQL và tăng tốc độ phản hồi của hệ thống. Trong WorkflowHub, Redis sẽ phục vụ 4 mục tiêu chính:

1.  **Session Management:** Lưu trữ JWT/Session của người dùng.
2.  **Organization/Project Settings:** Caching các cấu hình JSON thường xuyên truy cập.
3.  **Rate Limiting:** Giới hạn số lượng request (đặc biệt là API gọi đến AI).
4.  **AI Chat Context:** Lưu trữ tạm thời các tin nhắn gần nhất để gửi cho Agent (Context Window).

---

## 🔑 Key Naming Convention
Để đảm bảo an toàn trong môi trường Multi-tenant, tất cả các key (trừ session global) đều phải bao gồm `organization_id`.

**Pattern:** `wh:{tenant_id}:{entity}:{id}`

| Loại dữ liệu | Key Pattern | TTL (Thời gian sống) |
| :--- | :--- | :--- |
| **User Session** | `wh:session:{user_id}` | 24 Hours |
| **Org Settings** | `wh:{org_id}:settings` | 1 Hour |
| **Project Details** | `wh:{org_id}:project:{project_id}` | 30 Minutes |
| **Rate Limit** | `wh:limit:{ip/user_id}:{endpoint}` | 1 Minute |
| **AI Context Buffer**| `wh:{org_id}:chat:{chat_id}:buffer` | 10 Minutes |

---

## 🚀 Cache Invalidation (Làm mới Cache)
Chúng se sử dụng chiến lược **Write-through** hoặc **Cache Aside**:
*   Khi Update dữ liệu trong MySQL (vd: Sửa project settings) -> Backend đồng thời thực hiện lệnh `DEL` hoặc `SET` lại key tương ứng trong Redis.

---

## 🤖 AI Context System (Redis đặc thù)
Để tiết kiệm chi phí và tăng tốc độ cho AI:
*   Thay vì mỗi lần Chat lại query toàn bộ lịch sử từ MySQL bảng `messages`.
*   Backend sẽ lưu 5-10 tin nhắn gần nhất vào Redis List (`buffer`).
*   Khi Agent cần context, Backend lấy thẳng từ Redis để gửi đi.
*   Chỉ khi kết thúc phiên chat hoặc Redis hết hạn, dữ liệu mới được "persist" hoàn toàn vào MySQL (nếu chưa lưu).

---

## 🛡️ Security
*   Redis sẽ được triển khai trong mạng nội bộ (Private Network), không public ra internet.
*   Sử dụng Password/ACL để phân quyền truy cập.

---

*Last Updated: 2026-02-11*
