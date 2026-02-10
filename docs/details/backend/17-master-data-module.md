# Master Data Module (Lookups)

**Version:** v1  
**Date:** 2026-02-11  
**Skills:** `backend-architect`, `nodejs-best-practices`

---

## 🎯 Overview

Master Data Module quản lý các bảng danh mục dùng chung (Lookup Tables) giúp hệ thống linh hoạt và có khả năng tùy biến cao cho từng Organization.

**Các thực thể quản lý:**
1. **Categories:** Danh mục cho Projects, Tasks, Documents.
2. **Tags:** Nhãn dán cho Issues, Tasks.
3. **Roles & Permissions:** Định nghĩa các nhóm quyền.
4. **Workflow Statuses:** Trạng thái động cho quy trình làm việc.

---

## 📂 Module Structure (Shared Implementation)

Vì các bảng này có logic CRUD tương tự nhau, chúng ta có thể gộp vào một module `master-data` hoặc tách nhỏ nhưng dùng chung Base Repository/Service.

```text
src/modules/master-data/
├── controllers/
├── services/
├── repositories/
├── models/ (Category.js, Tag.js, Role.js, etc.)
└── index.js
```

---

## 🏗️ Technical Implementation: Polymorphic Categories

Để tránh tạo quá nhiều bảng, chúng ta sử dụng một bảng `categories` duy nhất với cột `type`:

| id | organization_id | name | type | color |
| :--- | :--- | :--- | :--- | :--- |
| 1 | org_a | "Bug" | `issue` | #ff0000 |
| 2 | org_a | "Feature" | `project` | #00ff00 |

---

## 🛡️ Business Rules

### 1. Tenant Isolation
Mọi danh mục (Categories, Tags) phải được gắn với `organization_id`. 
* **Global Data:** Có thể có một số danh mục mặc định của hệ thống (`organization_id = NULL`). User chỉ được xem, không được sửa.

### 2. Cascading Behavior
* Khi xóa một `Category`, cần xử lý các bản ghi đang sử dụng category đó (như Tasks). 
* **Strategy:** Set NULL hoặc chuyển sang "Uncategorized".

### 3. Permissions
* Chỉ **Admin/Owner** mới có quyền thêm/sửa/xóa Master Data của tổ chức.

---

## 🎮 Key Endpoints (Generic Pattern)

`GET /api/organizations/:orgId/master-data/:type`

**Types:** `categories`, `tags`, `roles`, `statuses`.

---

*Last Updated: 2026-02-11*
