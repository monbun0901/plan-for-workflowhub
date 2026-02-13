# Phase 1: Core System (Human-Centric Foundation)

**Status:** ✅ Backend Complete  
**Timeline:** Dự kiến 4-6 tuần → Actual: ~2 tuần (backend)

---

## 🌟 Vision
Xây dựng nền tảng vững chắc nhất cho một hệ thống SaaS Enterprise. Giai đoạn này tập trung vào việc quản lý dữ liệu chính xác, cô lập tuyệt đối giữa các tổ chức (Multi-tenant) và cung cấp bộ công cụ quản lý dự án mạnh mẽ cho con người.

---

## 🏗️ Core Pillars (Các trụ cột chính)

### 1. Identity & Tenant Isolation (Định danh & Cô lập)
*   **Authentication:** Hệ thống đăng ký/đăng nhập chuẩn bảo mật với JWT và Refresh Token.
*   **Multi-tenant Engine:** Cơ chế lọc dữ liệu theo `organization_id` ở mức thấp nhất của Database (Repository layer).
*   **RBAC (Role-Based Access Control):** Phân quyền chi tiết: Owner, Admin, Member.

### 2. Project Management (Quản lý dự án)
*   **Projects:** Không gian làm việc cho các nhóm.
*   **GitHub-Style Issues:** Hệ thống theo dõi vấn đề, lỗi và yêu cầu tính năng với Labels, Status và Milestone.
*   **Task Execution:** Chia nhỏ Issue thành các nhiệm vụ cụ thể có Deadline và người chịu trách nhiệm (Assignee).

### 3. Collaboration (Cộng tác)
*   **Comments:** Thảo luận trực tiếp trên Issues và Tasks.
*   **Real-time Notifications:** Thông báo khi có thay đổi trạng thái hoặc được nhắc tên.

---

## 📈 Strategic Value
*   **Data Integrity:** Đảm bảo dữ liệu được tổ chức khoa học, không bao giờ bị rò rỉ giữa các khách hàng.
*   **Productivity:** Cung cấp ngay giá trị cho người dùng như một công cụ quản lý công việc chuyên nghiệp (giống Trello/Jira/GitHub).
*   **Foundation for AI:** Tạo ra các "vết chân dữ liệu" (Data footprints) để AI có thể học và phân tích ở Phase 2.

---

*“Phase 1 là bộ khung xương vững chắc để WorkflowHub có thể đứng vững.”*

---

*Last Updated: 2026-02-11*
