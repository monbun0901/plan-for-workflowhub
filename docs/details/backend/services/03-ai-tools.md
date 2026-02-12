# AI Tools Specification

**Status:** 🏗️ Phase 2 Design  
**Core Responsibility:** Định nghĩa giao diện (Interface) để AI Agents có thể tương tác trực tiếp với dữ liệu trong hệ thống.

---

## 🛠️ Tool Definition Format (OpenAI Style)

Mỗi công cụ (Tool) sẽ được định nghĩa là một JSON Schema để AI hiểu lúc nào cần dùng và cung cấp tham số gì.

### Cấu trúc chung:
```json
{
  "type": "function",
  "function": {
    "name": "tên_hàm",
    "description": "Mô tả chi tiết để AI biết khi nào nên dùng hàm này",
    "parameters": {
      "type": "object",
      "properties": {
        "params_1": { "type": "string", "description": "..." }
      },
      "required": ["params_1"]
    }
  }
}
```

---

## 📋 Core Tool List (Danh mục công cụ lõi)

### 1. Project Management Tools
Dùng cho Agent vai trò PM hoặc Developer để điều phối công việc.

*   **`create_task`**: Tạo nhiệm vụ mới.
    *   Params: `project_id`, `name`, `priority`, `due_date`.
*   **`update_task_status`**: Thay đổi trạng thái task.
    *   Params: `task_id`, `status_id`.
*   **`assign_issue`**: Chỉ định người xử lý issue.
    *   Params: `issue_id`, `user_id`.

### 2. Knowledge & Data Tools
Dùng để truy xuất dữ liệu ngoài phạm vi RAG cung cấp.

*   **`get_document_content`**: Đọc toàn bộ nội dung của một tài liệu cụ thể.
    *   Params: `document_id`.
*   **`list_project_issues`**: Lấy danh sách issue của một dự án.
    *   Params: `project_id`, `status`.

---

## 🛡️ Execution & Security (Vô cùng quan trọng)

AI Gateway **KHÔNG** tự thực thi hàm. Nó chỉ trả về yêu cầu: *"Tôi muốn gọi hàm X với tham số Y"*. 

**Quy trình thực thi an toàn tại Backend:**

1.  **Permission Check:** Trước khi chạy hàm `update_task_status`, Backend phải kiểm tra `req.organization.permissions` xem người dùng (người đang chat) có quyền đó không.
2.  **Input Validation:** Sử dụng **Zod** để validate tham số AI gửi về (tránh việc AI gửi tham số linh tinh làm crash server).
3.  **Audit Log:** Mọi hành động AI thực hiện thông qua Tool phải được ghi lại: *"Agent PM đã tạo Task X dựa trên yêu cầu của User Y"*.

---

## 📝 Example: `create_task` Tool Schema

```json
{
  "type": "function",
  "function": {
    "name": "create_workflow_task",
    "description": "Tạo một task mới trong project hiện tại khi có yêu cầu từ người dùng hoặc theo quy trình",
    "parameters": {
      "type": "object",
      "properties": {
        "project_id": { "type": "string", "format": "uuid" },
        "name": { "type": "string", "description": "Tiêu đề ngắn gọn của task" },
        "priority": { "type": "string", "enum": ["low", "medium", "high", "urgent"] },
        "description": { "type": "string", "description": "Mô tả chi tiết công việc" }
      },
      "required": ["project_id", "name"]
    }
  }
}
```

---

## 💡 AI-Prompting Tip for Tools
Luôn đính kèm hướng dẫn này vào System Prompt:
> "Khi bạn thực hiện một hành động (Tool Call), hãy luôn giải thích cho người dùng biết bạn đang làm gì và tại sao bạn làm vậy dựa trên tài liệu nội bộ."

---

*Last Updated: 2026-02-11*
