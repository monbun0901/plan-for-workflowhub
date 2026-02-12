# Organizations Module

**Version:** v1  
**Date:** 2026-02-11  
**Skills:** `backend-architect`, `nodejs-best-practices`

---

## 🎯 Overview

Organizations module là cốt lõi của hệ thống **Multi-tenant**. Nó quản lý thực thể cao nhất mà tất cả các dữ liệu khác (Projects, Tasks, Members) đều phải thuộc về.

**Pattern:** Controller → Service → Repository

---

## 📂 Module Structure (Follows Standard Pattern)

```text
src/modules/organizations/
├── controllers/
│   └── organization.controller.js
├── services/
│   └── organization.service.js
├── repositories/
│   └── organization.repository.js
├── models/
│   └── Organization.model.js
├── dtos/
│   ├── create-org.dto.js
│   └── update-org.dto.js
└── index.js
```

---

## 🗄️ Repository Layer (Logic cô lập dữ liệu)

```javascript
/**
 * @class OrganizationRepository
 */
class OrganizationRepository {
  /**
   * Lấy thông tin tổ chức theo ID
   * @param {string} id 
   */
  async findById(id) {
    return Organization.findByPk(id);
  }

  /**
   * Tạo tổ chức mới
   * @param {Object} data 
   */
  async create(data) {
    return Organization.create(data);
  }

  /**
   * Lấy danh sách tổ chức mà User là thành viên
   * @param {string} userId 
   */
  async listByUser(userId) {
    return Organization.findAll({
      include: [{
        model: Member,
        where: { user_id: userId }
      }]
    });
  }
}
```

---

## ⚙️ Service Layer (Logic nghiệp vụ)

```javascript
/**
 * @class OrganizationService
 */
class OrganizationService {
  constructor(orgRepo, memberRepo) {
    this.orgRepo = orgRepo;
    this.memberRepo = memberRepo;
  }

  /**
   * Khi tạo Org mới, người tạo tự động trở thành Owner
   */
  async createOrganization(userId, data) {
    const org = await this.orgRepo.create(data);
    
    // Tạo Member đầu tiên với Role là Owner
    await this.memberRepo.create({
      organization_id: org.id,
      user_id: userId,
      role: 'owner',
      status: 'active'
    });

    return org;
  }
}
```

---

## 🎮 Key Endpoints

| Method | Endpoint | Description | Permission |
| :--- | :--- | :--- | :--- |
| `GET` | `/api/organizations` | List orgs of current user | `authenticated` |
| `POST` | `/api/organizations` | Create new organization | `authenticated` |
| `GET` | `/api/organizations/:id` | Get org details | `member` |
| `PUT` | `/api/organizations/:id` | Update settings/details | `admin/owner` |
| `DELETE`| `/api/organizations/:id` | Archive/Delete org | `owner` |

---

## 🛡️ Business Rules
1. **Owner Protection:** Một tổ chức phải luôn có ít nhất một Owner. Không được phép xóa Owner duy nhất.
2. **Slug Uniqueness:** Mỗi tổ chức nên có một `slug` duy nhất (ví dụ: `workflowhub.ai/my-company`) để định danh trên URL.
3. **Settings Validation:** Các cấu hình trong cột `settings` (JSON) phải được validate chặt chẽ theo schema quy định.

---

*Last Updated: 2026-02-11*
