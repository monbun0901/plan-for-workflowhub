# chats Table

**Type:** AI Chat Sessions  
**Tenant Isolation:** ✅ Required (`organization_id`)

---

## 📋 Schema

```sql
CREATE TABLE chats (
  id              VARCHAR(36) PRIMARY KEY,
  organization_id VARCHAR(36) NOT NULL REFERENCES organizations(id),
  project_id      VARCHAR(36) REFERENCES projects(id),
  
  user_id         VARCHAR(36) NOT NULL REFERENCES users(id),
  agent_id        VARCHAR(36) REFERENCES agents(id),
  
  title           VARCHAR(200),
  status          ENUM('active', 'archived') DEFAULT 'active',
  
  -- Context
  context_type    ENUM('general', 'project', 'issue', 'task', 'document'),
  context_id      VARCHAR(36),
  
  created_at      TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at      TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  
  INDEX idx_user (user_id),
  INDEX idx_org (organization_id)
);
```

---

## 🔗 Associations (Sequelize)

```typescript
// models/chat.model.ts
Chat.belongsTo(User, {
  foreignKey: 'user_id',
  as: 'user'
});

Chat.belongsTo(Agent, {
  foreignKey: 'agent_id',
  as: 'agent'
});

Chat.belongsTo(Project, {
  foreignKey: 'project_id',
  as: 'project'
});

Chat.hasMany(Message, {
  foreignKey: 'chat_id',
  as: 'messages'
});
```

**Explanation:**
- `belongsTo(User)` - Chat session của user
- `belongsTo(Agent)` - Agent được chat với (optional: có thể multi-agent)
- `belongsTo(Project)` - Project context (optional)
- `hasMany(Message)` - Chat chứa nhiều messages

---

## 📝 Fields Explanation - Context System

| Field | Description | Example |
|-------|-------------|---------|
| `context_type` | What is being discussed | 'issue' (discussing issue #42) |
| `context_id` | Entity ID | issue.id = 'uuid-123' |
| `title` | Chat summary | "Fix login bug discussion" |

**Context Examples:**
- `general` - Open-ended chat, `context_id: null`
- `project` - Project overview, `context_id: project.id`
- `issue` - Discussing specific issue, `context_id: issue.id`
- `document` - Q&A about doc, `context_id: document.id`

---

## 🎯 Common Queries

### Get user's recent chats

```typescript
const chats = await Chat.findAll({
  where: {
    user_id: userId,
    organization_id: orgId,
    status: 'active'
  },
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

### Get chat with full history

```typescript
const chat = await Chat.findOne({
  where: { id: chatId, organization_id: orgId },
  include: [{
    model: Message,
    as: 'messages',
    order: [['created_at', 'ASC']]
  }]
});
```

---

## 📚 Related Tables

- **messages** - Chat messages (separate table)
- **agents** - AI agent in conversation
- **users** - Chat owner

---

*Last Updated: 2026-02-11*
