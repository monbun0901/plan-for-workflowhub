# Infrastructure: Docker Stack Guide

**Version:** v1.0.0
**Status:** 🏗️ Infrastructure Plan

Để đảm bảo hiệu năng tối đa (1ms latency) và khả năng lưu trữ không giới hạn cho Phase 2, hệ thống WorkflowHub sẽ sử dụng bộ Stack Docker cục bộ.

---

## 🏗️ Services Overview

| Service | Technology | Port | Purpose |
| :--- | :--- | :--- | :--- |
| **Primary DB** | MySQL 8.0 | `3306` | Lưu trữ dữ liệu quan hệ, business logic, users. |
| **Cache DB** | Redis 7 | `6379` | Cache session, rate limiting, and AI context buffer. |
| **Vector DB** | ChromaDB | `8001` | Trí nhớ của AI (RAG), lưu trữ embeddings. |
| **DB Admin** | phpMyAdmin | `8888` | Giao diện quản lý GUI cho MySQL. |

---

## 📂 Configuration

File `docker-compose.yaml` được đặt tại thư mục gốc của dự án.

### Volumes (Data Persistence)
Dữ liệu được lưu trữ bền vững ngay cả khi container bị xóa thông qua Docker Volumes:
- `mysql_data`: Dữ liệu tài khoản, dự án.
- `redis_data`: Dữ liệu đệm.
- `chromadb_data`: Cơ sở dữ liệu tri thức của AI.

### Security Note
Mật khẩu và các thông tin nhạy cảm được quản lý qua biến môi trường (`.env`).

---

## 🚀 Commands

### 1. Khởi chạy hệ thống
```bash
docker-compose up -d
```

### 2. Kiểm tra trạng thái
```bash
docker-compose ps
```

### 3. Dừng hệ thống (giữ lại dữ liệu)
```bash
docker-compose stop
```

### 4. Xóa hoàn toàn (bao gồm cả dữ liệu)
```bash
docker-compose down -v
```

---

*Last Updated: 2026-02-11*
