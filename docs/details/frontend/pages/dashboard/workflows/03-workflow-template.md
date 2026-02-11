# Workflow Template Management Page

**Status:** 🔴 Coming (Phase 2)
**Route:** `/workflows/templates/:id`
**Layout:** [Shell Layout](../../../layouts/01-shell-layout.md)

---

## 📐 Wireframe (Concept)

```
┌─ Sidebar ─┬──────────────────────────────────────┐
│            │  Header: Workflows > Templates > Name │
│            ├──────────────────────────────────────┤
│            │                                      │
│            │  ┌─ Template Info ───────────────┐   │
│            │  │ 📋 "Sprint Planning Template"  │   │
│            │  │ Category: Agile               │   │
│            │  │ Steps: 5  │  Usage: 42 times  │   │
│            │  └───────────────────────────────┘   │
│            │                                      │
│            │  ┌─ Preview (read-only flow) ────┐   │
│            │  │  Trigger → Action → Condition  │   │
│            │  │  → Action → End                │   │
│            │  └───────────────────────────────┘   │
│            │                                      │
│            │  ┌─ Actions ─────────────────────┐   │
│            │  │ [Use This Template]             │   │
│            │  │ [Edit Template] (admin only)    │   │
│            │  │ [Delete] (admin only)           │   │
│            │  └───────────────────────────────┘   │
└────────────┴──────────────────────────────────────┘
```

---

## ✅ Chức năng (Roadmap)

| Feature | Status | Mô tả |
|---------|--------|--------|
| Template detail view | 🔴 Coming | Info + step preview |
| "Use Template" → create workflow | 🔴 Coming | Clone to own workflows |
| Edit template (admin) | 🔴 Coming | Modify steps |
| Delete template (admin) | 🔴 Coming | Remove template |
| Template categories | 🔴 Coming | Agile, DevOps, Custom |
| Template marketplace | 🔴 Coming | Share/import templates |

---

## 📡 API Endpoints

| Method | Endpoint | Mô tả |
|--------|----------|--------|
| `GET` | `/workflow-templates/:id` | Template detail |
| `POST` | `/workflows/from-template/:id` | Create from template |
| `PUT` | `/workflow-templates/:id` | Edit template |
| `DELETE` | `/workflow-templates/:id` | Delete template |

---

*Last Updated: 2026-02-11*
