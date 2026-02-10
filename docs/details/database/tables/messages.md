# messages Table

**Type:** Chat Message History  
**Tenant Isolation:** ✅ Required (`organization_id`)

---

## 📋 Schema

```sql
CREATE TABLE messages (
  id              VARCHAR(36) PRIMARY KEY,
  chat_id         VARCHAR(36) NOT NULL REFERENCES chats(id),
  organization_id VARCHAR(36) NOT NULL REFERENCES organizations(id),
  
  -- Role định danh người gửi
  role            ENUM('user', 'assistant', 'system') NOT NULL,
  content         TEXT NOT NULL,
  
  -- Metadata trả về từ Agent (Optional)
  -- Lưu các thông tin bổ sung như citations, tool calls, hoặc kết quả xử lý
  metadata        JSON,                          -- { "sources": [...], "latency": 1200 }
  
  created_at      TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  
  INDEX idx_chat (chat_id),
  INDEX idx_org (organization_id)
);
```

---

## 🎯 Purpose
Lưu trữ lịch sử hội thoại trong các phiên Chat. Backend chỉ đóng vai trò lưu lại những gì User nói và những gì Agent phản hồi để hiển thị lại lịch sử cho người dùng.

---

## 🔗 Associations (Sequelize)

```typescript
// models/message.model.ts
Message.belongsTo(Chat, {
  foreignKey: 'chat_id',
  as: 'chat'
});

Message.belongsTo(Organization, {
  foreignKey: 'organization_id',
  as: 'organization'
});
```

---

## 📝 Fields Explanation

| Field | Description | Example |
|-------|-------------|---------|
| `role` | Vai trò người gửi | user, assistant (AI), system |
| `content` | Nội dung tin nhắn | "Dự án này đang gặp vấn đề gì?" |
| `metadata` | Dữ liệu bổ sung từ Agent | Lưu danh sách tài liệu tham khảo (citations), thông tin kỹ thuật... |

---

## � Metadata Structure (Ví dụ)

```json
{
  "sources": [
    {
      "document_id": "uuid-123",
      "title": "API Guide",
      "similarity": 0.95
    }
  ],
  "agent_version": "v1.2",
  "processing_time_ms": 1500
}
```

---

## 🎯 Common Queries

### Get chat history

```typescript
const messages = await Message.findAll({
  where: { chat_id: chatId, organization_id: orgId },
  order: [['created_at', 'ASC']]
});
```

---

## 📚 Related Tables

- **chats** - Phiên hội thoại cha.
- **agents** - Agent thực hiện phản hồi (thông qua bảng chats).

---

*Last Updated: 2026-02-11*
