# Backend Architecture

**Version:** v1  
**Date:** 2026-02-11  
**Skills:** `backend-architect`, `nodejs-best-practices`

---

## 🎯 Overview

WorkflowHub backend sử dụng **Modular Monolith** architecture với **Layered Pattern**.

---

## 🏗️ Technical Stack (Backend)

*   **Runtime:** Node.js (`type: module` in package.json)
*   **Compiler:** **Babel** (Xử lý các tính năng nâng cao và đảm bảo tương thích)
*   **Framework:** Express.js (OOP Style)
*   **Database:** MySQL + **Sequelize ORM**
*   **Migrations:** Bắt buộc dùng định dạng **CommonJS (.cjs)** để tương thích với Sequelize CLI trong môi trường ESM.
*   **Validation:** Zod
*   **Documentation:** JSDoc (Mandatory)

---

## 📂 Feature-Based Directory Structure

Cấu trúc này tối ưu cho việc mở rộng theo từng tính năng (Feature-based) mà vẫn đảm bảo tương thích hoàn toàn với `sequelize-cli`.

```text
workflowhub-backend/
├── .babelrc                    # Cấu hình Babel
├── .sequelizerc                # Cấu hình đường dẫn cho Sequelize CLI
├── package.json                # Có "type": "module"
├── src/
│   ├── app.js                  # ESM (import/export)
│   ├── modules/
│   │   ├── ...
│   │   └── users/
│   │       └── models/
│   │           └── User.model.js # ESM
│   └── database/
│       ├── migrations/         # CÁC FILE .cjs (CommonJS)
│       │   └── 20240101-create-users.cjs
│       ├── seeders/            # CÁC FILE .cjs
│       └── index.js            # Boilerplate nạp model (ESM)
```

---

## 🛠️ Sequelize Lifecycle & `.sequelizerc`

Để `sequelize-cli` không bị rối khi chúng ta đặt model trong từng module, ta cấu hình `.sequelizerc` như sau:

```javascript
// .sequelizerc
const path = require('path');

module.exports = {
  'config': path.resolve('src', 'config', 'database.js'),
  'models-path': path.resolve('src', 'database', 'models'), // Điểm nạp tập trung
  'seeders-path': path.resolve('src', 'database', 'seeders'),
  'migrations-path': path.resolve('src', 'database', 'migrations')
};
```

**Chiến lược nạp mô hình (Model Discovery):**
Trong `src/database/index.js`, chúng ta sẽ viết một hàm tự động quét tất cả thư mục `modules/*/models/*.js` để đăng ký với Sequelize.

---

## 🏛️ OOP Class Pattern (Chuẩn hóa)

Mọi thành phần phải được viết dưới dạng **Class** và có **JSDoc** đầy đủ.

```javascript
/**
 * @class ProjectService
 * @description Xử lý các logic nghiệp vụ liên quan đến dự án
 */
class ProjectService {
  /**
   * @param {ProjectRepository} projectRepo - Repository nạp qua DI hoặc khởi tạo
   */
  constructor(projectRepo) {
    this.projectRepo = projectRepo;
  }

  /**
   * @async
   * @method createProject
   * @param {string} organizationId - ID của tổ chức (Multi-tenant)
   * @param {Object} data - Dữ liệu dự án từ Controller
   * @returns {Promise<Object>} Trả về project đã tạo
   * @throws {AppError} 400 nếu dữ liệu không hợp lệ
   */
  async createProject(organizationId, data) {
    // Logic xử lý...
  }
}

export default ProjectService;
```

### With DI (tsyringe - Advanced)

```typescript
// apps/api/src/modules/projects/services/project.service.ts
import { injectable, inject } from 'tsyringe';
import { ProjectRepository } from '../repositories/project.repository';

@injectable()
export class ProjectService {
  constructor(
    @inject('ProjectRepository') 
    private readonly projectRepo: ProjectRepository
  ) {}
  
  // ...methods
}
```

**Recommendation:** Start without DI for MVP, add later if needed.

---

## ⚡ Best Practices Checklist

### Module Design
- [ ] **Single Responsibility** - Each module handles one domain
- [ ] **Clear Boundaries** - No cross-module direct imports (use services)
- [ ] **Thin Controllers** - Controller chỉ handle HTTP, không có business logic
- [ ] **Fat Services** - Business logic tập trung ở Service layer
- [ ] **Repository Abstraction** - Data access isolated trong Repository

### Code Quality
- [ ] **TypeScript Strict** - Enable strict mode
- [ ] **JSDoc** - Document all public methods
- [ ] **Error Handling** - Centralized error middleware
- [ ] **Validation** - Zod schemas cho all inputs
- [ ] **No Magic Values** - Constants centralized

### Security
- [ ] **Tenant Isolation** - All queries filter by `organization_id`
- [ ] **Input Validation** - Zod validation on all DTOs
- [ ] **SQL Injection** - Use Sequelize parameterized queries
- [ ] **XSS Protection** - Sanitize outputs
- [ ] **Rate Limiting** - Applied on auth + sensitive endpoints

---

## 🔗 Module Dependencies

```
┌─────────────────────────────────────────────────┐
│              Module Dependency Graph             │
├─────────────────────────────────────────────────┤
│                                                 │
│   auth ───┐                                     │
│           │                                     │
│   tenant ─┴──▶ projects ──▶ issues ──▶ tasks  │
│                    │                            │
│                    ├──▶ documents ──▶ RAG      │
│                    │                            │
│                    └──▶ workflows              │
│                                                 │
│   agents ───────────▶ chat ──▶ RAG             │
│                                                 │
└─────────────────────────────────────────────────┘
```

**Rule:** Higher-level modules can import lower-level modules, not reverse.

---

## 🏗️ ESM Model vs CJS Migration (Practical Samples)

Đây là ví dụ đối chiếu giữa cách viết Model (Hiện đại - ESM) và Migration (Truyền thống - CJS).

### 1. Migration mẫu (src/database/migrations/xxx-create-users.cjs)
Sử dụng `.cjs` và `module.exports` để Sequelize CLI có thể đọc được.

```javascript
'use strict';

/** @type {import('sequelize-cli').Migration} */
module.exports = {
  async up(queryInterface, Sequelize) {
    await queryInterface.createTable('users', {
      id: {
        allowNull: false,
        primaryKey: true,
        type: Sequelize.UUID,
        defaultValue: Sequelize.UUIDV4
      },
      email: {
        type: Sequelize.STRING(255),
        allowNull: false,
        unique: true
      },
      created_at: {
        allowNull: false,
        type: Sequelize.DATE
      },
      updated_at: {
        allowNull: false,
        type: Sequelize.DATE
      }
    });
  },

  async down(queryInterface) {
    await queryInterface.dropTable('users');
  }
};
```

### 2. Model mẫu (src/modules/users/models/User.model.js)
Sử dụng ESM `import/export` và định dạng Class OOP.

```javascript
import { Model, DataTypes } from 'sequelize';

/**
 * @class User
 * @extends Model
 * @description Đại diện cho tài khoản người dùng trong hệ thống
 */
class User extends Model {
  /**
   * Khởi tạo schema cho model
   * @param {Sequelize} sequelize - Instance của Sequelize
   */
  static init(sequelize) {
    return super.init({
      id: {
        type: DataTypes.UUID,
        defaultValue: DataTypes.UUIDV4,
        primaryKey: true
      },
      email: {
        type: DataTypes.STRING,
        allowNull: false,
        unique: true,
        validate: { isEmail: true }
      }
    }, {
      sequelize,
      modelName: 'User',
      tableName: 'users',
      underscored: true, // Chuyển camelCase sang snake_case trong DB
      timestamps: true
    });
  }

  /**
   * Định nghĩa các mối quan hệ (Associations)
   * @param {Object} models - Danh sách tất cả các models đã nạp
   */
  static associate(models) {
    this.hasMany(models.Member, { foreignKey: 'user_id', as: 'memberships' });
  }
}

export default User;
```

---

## 📚 Related Documents

- [02-authentication.md](02-authentication.md) - Auth implementation
- [03-multi-tenant.md](03-multi-tenant.md) - Tenant isolation  
- [04-projects-module.md](04-projects-module.md) - Reference implementation
- [README.md](README.md) - Backend overview

---

*Document Version: 1.0*  
*Last Updated: 2026-02-11*
