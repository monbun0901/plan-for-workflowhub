# chats Table

**Type:** AI Chat Sessions  
**Tenant Isolation:** N/A (Single-Tenant)

---

## 📋 Schema

```sql
CREATE TABLE chats (
  id              VARCHAR(36) PRIMARY KEY,
  user_id         VARCHAR(36) NOT NULL REFERENCES users(id),
  agent_id        VARCHAR(36) NOT NULL REFERENCES agents(id), -- Chat với Agent nào?
  
  title           VARCHAR(200),
  status          ENUM('active', 'archived') DEFAULT 'active',
  
  -- Sợi dây liên kết (AI sẽ đọc dữ liệu từ Context này để trả lời)
  context_type    ENUM('workflow_instance', 'issue', 'task', 'document', 'project', 'general') DEFAULT 'general',
  context_id      VARCHAR(36), -- ID tương ứng của entity (vd: ID của Task #1)
  
  created_at      TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at      TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  
  INDEX idx_user (user_id),
  INDEX idx_context (context_type, context_id)
);
```

---

## 🔗 Associations (Sequelize)

```typescript
// models/chat.model.ts
Chat.belongsTo(User, { foreignKey: 'user_id', as: 'user' });
Chat.belongsTo(Agent, { foreignKey: 'agent_id', as: 'agent' });
Chat.hasMany(Message, { foreignKey: 'chat_id', as: 'messages' });
```

---

## 🎯 Common Queries

### Get user's recent chats

```typescript
const chats = await Chat.findAll({
  where: { user_id: userId, status: 'active' },
  include: [
    { model: Agent, as: 'agent' },
    { 
      model: Message, 
      as: 'messages',
      limit: 1,
      order: [['created_at', 'DESC']]
    }
  ],
  order: [['updated_at', 'DESC']],
  limit: 20
});
```

---

*Last Updated: 2026-02-15*
