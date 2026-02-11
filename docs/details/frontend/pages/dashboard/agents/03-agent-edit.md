# Edit Agent Page

**Status:** 🟡 Scale-up
**Route:** `/agents/:id/edit`
**Layout:** [CRUD Page Layout](../../../layouts/03-crud-page-layout.md)

---

## 📐 Wireframe

Giống `02-agent-create.md` nhưng pre-filled.

```
┌─ Sidebar ─┬──────────────────────────────────┐
│            │  Header: Agents > Edit             │
│            ├──────────────────────────────────┤
│            │     max-w-2xl (centered)          │
│            │                                  │
│            │  ┌─ Page Header ──────────────┐  │
│            │  │ "Edit Agent"       [Cancel] │  │
│            │  └────────────────────────────┘  │
│            │                                  │
│            │  ┌─ Form Card (pre-filled) ───┐  │
│            │  │  Name:     [DevBot v2   ]   │  │
│            │  │  Role:     [Coder ▼     ]   │  │
│            │  │  Model:    [Claude-3.5 ▼]   │  │
│            │  │                             │  │
│            │  │  System Prompt:             │  │
│            │  │  [You are a senior dev...]  │  │
│            │  │                             │  │
│            │  │  [Cancel]       [Save]      │  │
│            │  └─────────────────────────────┘ │
└────────────┴──────────────────────────────────┘
```

---

## ✅ Chức năng

| Feature | Status | Mô tả |
|---------|--------|--------|
| Pre-fill data | 🟡 Scale | Load existing configuration |
| Update model | 🟡 Scale | Change underlying LLM |
| Refine system prompt | 🟡 Scale | Edit instructions |
| History retention | 🔴 Coming | Keep chat history on model switch? |

---

## 🪝 Hooks

| Hook | Chức năng |
|------|----------|
| `useAgent(id)` | Fetch agent detail |
| `useUpdateAgent()` | Mutation + Toast |

## 📡 API Endpoints

| Method | Endpoint | Mô tả |
|--------|----------|--------|
| `GET` | `/agents/:id` | Agent detail |
| `PUT` | `/agents/:id` | Update agent |

---

*Last Updated: 2026-02-11*
