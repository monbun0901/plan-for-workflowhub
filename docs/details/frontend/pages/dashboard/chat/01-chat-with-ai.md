# Chat with AI Page

**Status:** 🔴 Coming (Phase 2)
**Route:** `/chat-with-ai`
**Layout:** [Shell Layout](../../../layouts/01-shell-layout.md)

---

## 📐 Wireframe (Concept)

```
┌─ Sidebar ─┬──────────────────────────────────────┐
│            │  Header: "AI Chat" + UserNav           │
│            ├──────────────────────────────────────┤
│            │                                      │
│            │  ┌─ Chat Sidebar ──┬─ Chat Area ──┐ │
│            │  │                 │               │ │
│            │  │ Conversations:  │  🤖 AI Agent  │ │
│            │  │ ● Chat 1        │               │ │
│            │  │ ○ Chat 2        │  Hello! How   │ │
│            │  │ ○ Chat 3        │  can I help?  │ │
│            │  │                 │               │ │
│            │  │ [+ New Chat]    │  👤 User      │ │
│            │  │                 │  Find bugs in │ │
│            │  │                 │  project Alpha│ │
│            │  │                 │               │ │
│            │  │                 │  🤖 AI Agent  │ │
│            │  │                 │  I found 3... │ │
│            │  │                 │               │ │
│            │  │                 ├───────────────┤ │
│            │  │                 │ [Type here...]│ │
│            │  │                 │        [Send] │ │
│            │  └─────────────────┴───────────────┘ │
└────────────┴──────────────────────────────────────┘
```

---

## ✅ Chức năng (Roadmap)

| Feature | Status | Mô tả |
|---------|--------|--------|
| Chat interface | 🔴 Coming | Message bubbles |
| Conversation list | 🔴 Coming | Sidebar with history |
| New conversation | 🔴 Coming | Create fresh thread |
| AI agent selection | 🔴 Coming | Choose persona |
| Context-aware (RAG) | 🔴 Coming | Querying project docs |
| Code generation | 🔴 Coming | Inline code blocks |
| Tool calling (create task) | 🔴 Coming | AI creates tasks/issues |
| Streaming responses | 🔴 Coming | Real-time token display |

---

## 📎 Dependencies (Phase 2)

Trang này phụ thuộc vào các backend modules:
- [08-agents-module.md](../../../../backend/08-agents-module.md) — AI Agent Personas
- [09-chat-module.md](../../../../backend/09-chat-module.md) — Chat Interface
- [12-rag-service.md](../../../../backend/12-rag-service.md) — Knowledge Engine
- [13-ai-gateway.md](../../../../backend/13-ai-gateway.md) — AI Orchestration

---

*Last Updated: 2026-02-11*
