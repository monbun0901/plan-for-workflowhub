# Create Agent Page

**Status:** 🟡 Scale-up
**Route:** `/agents/new`
**Layout:** [CRUD Page Layout](../../../layouts/03-crud-page-layout.md)

---

## 📐 Wireframe

```
┌─ Sidebar ─┬──────────────────────────────────┐
│            │  Header: Agents > New              │
│            ├──────────────────────────────────┤
│            │     max-w-2xl (centered)          │
│            │                                  │
│            │  ┌─ Page Header ──────────────┐  │
│            │  │ "Create New Agent" [Cancel] │  │
│            │  └────────────────────────────┘  │
│            │                                  │
│            │  ┌─ Form Card ────────────────┐  │
│            │  │                             │  │
│            │  │  Name:     [____________]   │  │
│            │  │  Role:     [Coder ▼     ]   │  │
│            │  │  Model:    [GPT-4o ▼    ]   │  │
│            │  │  Avatar:   [🤖 Upload   ]   │  │
│            │  │                             │  │
│            │  │  System Prompt:             │  │
│            │  │  [You are a senior dev...]  │  │
│            │  │  [_______________________]  │  │
│            │  │                             │  │
│            │  │  Knowledge Base:            │  │
│            │  │  [📁 Select Docs...    ]    │  │
│            │  │                             │  │
│            │  │  [Cancel]       [Create]    │  │
│            │  └─────────────────────────────┘ │
└────────────┴──────────────────────────────────┘
```

---

## ✅ Chức năng

| Feature | Status | Mô tả |
|---------|--------|--------|
| Name (required) | 🟡 Scale | Agent display name |
| Role selector | 🟡 Scale | Define core capabilities |
| Model selector | 🟡 Scale | Underlying LLM selection |
| Avatar upload | 🟡 Scale | Custom image |
| System Prompt | 🟡 Scale | Core instruction (textarea) |
| Knowledge Base link | 🔴 Coming | RAG source selection |
| Tools access | 🔴 Coming | Allow agent to use tools (search, git...) |
| Submit → Toast + redirect | 🟡 Scale | Redirect `/agents` |

---

## 🪝 Hooks

| Hook | Chức năng |
|------|----------|
| `useCreateAgentForm()` | Form state + Zod |
| `useCreateAgent()` | Mutation + Toast |
| `useModels()` | Fetch available LLM models |

## 📡 API Endpoints

| Method | Endpoint | Mô tả |
|--------|----------|--------|
| `POST` | `/agents` | Create new agent |
| `GET` | `/ai/models` | List available models |

---

*Last Updated: 2026-02-11*
