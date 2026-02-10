# Chat Module

**Version:** v1  
**Date:** 2026-02-11  
**Skills:** `backend-architect`

---

## 🎯 Overview

Chat module implements chat interface với AI agents + RAG (Retrieval-Augmented Generation).

**Status:** 📝 To Be Implemented  
**Pattern:** Follow [04-projects-module.md](04-projects-module.md) + RAG pipeline

---

## 📋 Implementation Guide

### Special Requirements

- **WebSocket support** - Real-time streaming responses
- **RAG pipeline** - Document retrieval + LLM
- **Message persistence** - Chat history storage
- **Agent orchestration** - Route to correct AI agent

### Chat Service with RAG

```typescript
// apps/api/src/modules/chat/services/chat.service.ts
export class ChatService {
  constructor(
    private readonly chatRepo: ChatRepository,
    private readonly redis: RedisService,
    private readonly aiGateway: AIGateway // Registry-based caller
  ) {}
  
  /**
   * Send message with Redis Sliding Window Context
   */
  async sendMessage(organizationId: string, chatId: string, message: string) {
    const contextKey = `wh:${organizationId}:chat:${chatId}:buffer`;

    // 1. Get recent context from Redis (Fast)
    let history = await this.redis.lrange(contextKey, 0, -1);
    
    // 2. If Redis is empty, refill from MySQL (Cache Refill)
    if (history.length === 0) {
      const dbHistory = await this.chatRepo.getRecentMessages(chatId, 10);
      history = dbHistory.map(m => JSON.stringify({ r: m.role, c: m.content }));
      if (history.length > 0) {
        await this.redis.rpush(contextKey, ...history);
      }
    }

    // 3. Call AI Gateway with history + new message
    const response = await this.aiGateway.callAgent({
      chatId,
      history: history.map(h => JSON.parse(h)),
      message
    });

    // 4. Persistence (Parallel)
    await Promise.all([
      // Multi-tenant save in MySQL
      this.chatRepo.saveMessage({ 
        chat_id: chatId, 
        organization_id: organizationId,
        content: message, 
        role: 'user' 
      }),
      this.chatRepo.saveMessage({
        chat_id: chatId,
        organization_id: organizationId,
        content: response.text,
        role: 'assistant',
        metadata: { sources: response.sources }
      }),
      // Update Redis Sliding Window
      this.redis.rpush(contextKey, 
        JSON.stringify({ r: 'u', c: message }),
        JSON.stringify({ r: 'a', c: response.text })
      ),
      this.redis.ltrim(contextKey, -10, -1) // Keep last 10
    ]);

    return response;
  }
}
```

### WebSocket Integration

```typescript
// apps/api/src/modules/chat/websocket/chat.gateway.ts
import { Server, Socket } from 'socket.io';

export class ChatGateway {
  async handleMessage(socket: Socket, data: { chatId: string; message: string }) {
    const organizationId = socket.data.organizationId;
    
    // Stream response
    const stream = await this.chatService.sendMessageStream(
      organizationId,
      data.chatId,
      data.message
    );
    
    for await (const chunk of stream) {
      socket.emit('message_chunk', { chunk });
    }
    
    socket.emit('message_complete');
  }
}
```

---

## 🔒 Security Considerations

- ✅ **Tenant isolation in RAG** - Only search org's documents
- ✅ **WebSocket auth** - Verify JWT on connection
- ✅ **Rate limiting** - Prevent AI abuse
- ✅ **Content filtering** - PII detection, harmful content

---

## 📚 Related Documents

- [07-documents-module.md](07-documents-module.md) - Vector DB integration
- [08-agents-module.md](08-agents-module.md) - AI agent selection
- [04-projects-module.md](04-projects-module.md) - Base pattern

---

*Status: Placeholder - Complex module requiring RAG + WebSocket*
