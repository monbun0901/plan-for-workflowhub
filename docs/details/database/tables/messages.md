# messages Table

**Type:** Chat Message History  
**Tenant Isolation:** N/A (Single-Tenant)

---

## 📋 Schema

```sql
CREATE TABLE messages (
  id              VARCHAR(36) PRIMARY KEY,
  chat_id         VARCHAR(36) NOT NULL REFERENCES chats(id),
  
  -- Vai trò trong hội thoại (BẮT BUỘC để AI hiểu ngữ cảnh)
  role            ENUM('user', 'assistant', 'system') NOT NULL, 
  content         TEXT NOT NULL,
  
  -- Metadata trả về từ Agent (Optional)
  metadata        JSON,                          -- { "sources": [...], "latency": 1200 }
  
  created_at      TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  
  INDEX idx_chat (chat_id)
);
```

---

## 🔗 Associations (Sequelize)

```typescript
// models/message.model.ts
Message.belongsTo(Chat, { foreignKey: 'chat_id', as: 'chat' });
```

---

## 🎯 Common Queries

### Get chat history

```typescript
const messages = await Message.findAll({
  where: { chat_id: chatId },
  order: [['created_at', 'ASC']]
});
```

---

*Last Updated: 2026-02-15*
