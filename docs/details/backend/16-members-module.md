# Members Module

**Version:** v1  
**Date:** 2026-02-11  
**Skills:** `backend-architect`, `nodejs-best-practices`

---

## 🎯 Overview

Members module quản lý mối quan hệ giữa **User** và **Organization**. Đây là nơi định nghĩa quyền hạn (RBAC) của một người trong một không gian làm việc cụ thể.

---

## 📂 Module Structure

```text
src/modules/members/
├── controllers/
│   └── member.controller.js
├── services/
│   └── member.service.js
├── repositories/
│   └── member.repository.js
├── models/
│   └── Member.model.js
├── dtos/
│   ├── invite-member.dto.js
│   └── update-role.dto.js
└── index.js
```

---

## 🗄️ Repository Layer (Logic quản lý thành viên)

```javascript
/**
 * @class MemberRepository
 */
class MemberRepository {
  /**
   * Kiểm tra User có thuộc Org không
   */
  async getMember(organizationId, userId) {
    return Member.findOne({
      where: { organization_id: organizationId, user_id: userId }
    });
  }

  /**
   * Lấy danh sách tất cả thành viên của Org
   */
  async listByOrg(organizationId) {
    return Member.findAll({
      where: { organization_id: organizationId },
      include: ['User'] // Nạp thông tin User kèm theo
    });
  }
}
```

---

## ⚙️ Service Layer (Logic mời và phân quyền)

```javascript
/**
 * @class MemberService
 */
class MemberService {
  /**
   * Mời thành viên mới vào tổ chức
   */
  async inviteMember(organizationId, inviterId, email, role) {
    // 1. Kiểm tra quyền của người mời (phải là Admin/Owner)
    // 2. Tìm User theo email trong hệ thống
    // 3. Nếu User chưa có, tạo Record mời (Invitation) - Phase 1b
    // 4. Nếu User đã có, tạo bản ghi Member với status 'pending'
  }

  /**
   * Thay đổi Role của thành viên
   */
  async updateMemberRole(organizationId, memberId, newRole) {
    // Logic ngăn chặn việc hạ cấp Owner cuối cùng
  }
}
```

---

## 🎮 Key Endpoints

| Method | Endpoint | Description | Permission |
| :--- | :--- | :--- | :--- |
| `GET` | `/api/organizations/:orgId/members` | Danh sách thành viên | `member` |
| `POST` | `/api/organizations/:orgId/members` | Mời thành viên mới | `admin/owner` |
| `PATCH`| `/api/organizations/:orgId/members/:id`| Cập nhật Role/Status | `admin/owner` |
| `DELETE`| `/api/organizations/:orgId/members/:id`| Xóa thành viên khỏi Org | `admin/owner` |

---

## 🛡️ Business Rules
1. **Self-Exit:** Thành viên có quyền tự rời khỏi tổ chức trừ khi họ là Owner duy nhất.
2. **Role Hierarchy:** Chỉ có Owner mới có quyền thay đổi Role của Owner khác hoặc xóa Owner.
3. **Status Flow:** `pending` (được mời) -> `active` (đã chấp nhận) -> `inactive` (bị vô hiệu hóa).

---

*Last Updated: 2026-02-11*
