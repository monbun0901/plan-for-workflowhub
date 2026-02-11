# Workflow Editor Page

**Status:** 🟡 Scale-up
**Route:** `/workflows/:id/edit` (cũng dùng cho `/workflows/new`)
**Layout:** [Shell Layout](../../../layouts/01-shell-layout.md)

---

## 📐 Wireframe

```
┌─ Sidebar ──┬──────────────────────────────────────┐
│            │  Header: Workflows > "Workflow Name" │
│            ├──────────────────────────────────────┤
│            │                                      │
│            │  ┌─ Toolbar ────────────────────┐    │
│            │  │ [Save Draft] [Activate] [⋯]  │    │
│            │  └──────────────────────────────┘     │
│            │                                       │
│            │  ┌─ Flow Builder ───────────────┐     │
│            │  │                               │    │
│            │  │  ┌─────────┐                  │    │
│            │  │  │ Trigger │ (When issue...)  │    │
│            │  │  └────┬────┘                  │    │
│            │  │       │                       │    │
│            │  │  ┌────▼────┐                  │    │
│            │  │  │ Action 1│ (Assign to...)   │    │
│            │  │  └────┬────┘                  │    │
│            │  │       │                       │    │
│            │  │  ┌────▼────┐                  │    │
│            │  │  │Condition│ (If priority...) │    │
│            │  │  └──┬───┬──┘                  │    │
│            │  │     │   │                     │    │
│            │  │  ┌──▼┐ ┌▼──┐                  │    │
│            │  │  │Yes│ │No │                  │    │
│            │  │  └───┘ └───┘                  │    │
│            │  │                               │    │
│            │  │       [+ Add Step]            │    │
│            │  └───────────────────────────────┘    │
│            │                                       │
│            │  ┌─ Step Config Panel (right) ───┐   │
│            │  │ Selected: "Action 1"          │   │
│            │  │ Type: [Assign Task ▼]         │   │
│            │  │ To:   [Team Lead ▼]           │   │
│            │  └───────────────────────────────┘   │
└────────────┴──────────────────────────────────────┘
```

---

## ✅ Chức năng

| Feature | Status | Mô tả |
|---------|--------|--------|
| Visual flow builder | 🟡 Scale | Drag & drop nodes |
| Step types: Trigger | 🟡 Scale | Event-based (issue created, task moved...) |
| Step types: Action | 🟡 Scale | Auto-assign, notify, change status |
| Step types: Condition | 🟡 Scale | If/else branching |
| Step config panel | 🟡 Scale | Configure selected step |
| Save as Draft | 🟡 Scale | Persist without activating |
| Activate workflow | 🟡 Scale | Make workflow live |
| Test run (dry run) | 🔴 Coming | Simulate workflow |
| AI-suggested steps | 🔴 Coming | AI recommends next step |

---

## 🪝 Hooks

| Hook | Chức năng |
|------|----------|
| `useWorkflow(id)` | Fetch workflow detail |
| `useWorkflowEditor()` | State machine for builder |
| `useSaveWorkflow()` | Save mutation |
| `useActivateWorkflow()` | Activate mutation |

## 📡 API Endpoints

| Method | Endpoint | Mô tả |
|--------|----------|--------|
| `GET` | `/workflows/:id` | Load workflow |
| `PUT` | `/workflows/:id` | Save workflow |
| `POST` | `/workflows/:id/activate` | Activate |
| `POST` | `/workflows` | Create new |

---

*Last Updated: 2026-02-11*
