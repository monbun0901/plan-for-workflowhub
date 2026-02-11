# Workflow List Page

**Status:** 🟡 Scale-up
**Route:** `/workflows`
**Layout:** [Shell Layout](../../../layouts/01-shell-layout.md)

---

## 📐 Wireframe

```
┌─ Sidebar ─┬──────────────────────────────────────┐
│            │  Header: "Workflows" + UserNav         │
│            ├──────────────────────────────────────┤
│            │                                      │
│            │  ┌─ Page Header ────────────────┐    │
│            │  │ "Workflows"   [+ New Workflow]│    │
│            │  └──────────────────────────────┘    │
│            │                                      │
│            │  ┌─ Tabs ───────────────────────┐    │
│            │  │ [My Workflows] [Templates]    │    │
│            │  └──────────────────────────────┘    │
│            │                                      │
│            │  ┌─ Grid ───────────────────────┐    │
│            │  │ ┌──────────┐ ┌──────────┐    │    │
│            │  │ │🔄        │ │🔄        │    │    │
│            │  │ │Workflow 1│ │Workflow 2│    │    │
│            │  │ │Active    │ │Draft     │    │    │
│            │  │ │3 steps   │ │5 steps   │    │    │
│            │  │ └──────────┘ └──────────┘    │    │
│            │  └──────────────────────────────┘    │
│            │                                      │
│            │  ┌─ Templates Tab ──────────────┐    │
│            │  │ ┌──────────┐ ┌──────────┐    │    │
│            │  │ │📋 Sprint │ │📋 Deploy │    │    │
│            │  │ │Template  │ │Template  │    │    │
│            │  │ │[Use this]│ │[Use this]│    │    │
│            │  │ └──────────┘ └──────────┘    │    │
│            │  └──────────────────────────────┘    │
└────────────┴──────────────────────────────────────┘
```

---

## ✅ Chức năng

### Tab: My Workflows
| Feature | Status | Mô tả |
|---------|--------|--------|
| Workflow grid | 🟡 Scale | Cards: name, status, step count |
| Create Workflow → page | 🟡 Scale | `/workflows/new` |
| Filter by status | 🟡 Scale | Active / Draft / Archived |
| Duplicate workflow | 🟡 Scale | Clone existing |
| Delete workflow | 🟡 Scale | Confirm dialog |
| Empty state | 🟡 Scale | "No workflows yet" + CTA |

### Tab: Templates
| Feature | Status | Mô tả |
|---------|--------|--------|
| Template grid | 🟡 Scale | Pre-built workflows |
| "Use Template" → create | 🟡 Scale | Clone template → edit |
| Template preview | 🟡 Scale | Visual step preview |
| Custom template CRUD | 🔴 Coming | Create own templates |

---

## 🪝 Hooks

| Hook | Chức năng |
|------|----------|
| `useWorkflows(orgId)` | List workflows |
| `useWorkflowTemplates()` | List templates |
| `useDuplicateWorkflow()` | Clone mutation |

## 📡 API Endpoints

| Method | Endpoint | Mô tả |
|--------|----------|--------|
| `GET` | `/workflows?status=...` | List workflows |
| `GET` | `/workflow-templates` | List templates |
| `POST` | `/workflows/from-template/:templateId` | Create from template |
| `DELETE` | `/workflows/:id` | Delete workflow |

---

*Last Updated: 2026-02-11*
