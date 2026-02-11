# Document Editor Page

**Status:** 🟡 Scale-up
**Route:** `/documents/:id/edit`
**Layout:** [Shell Layout](../../../layouts/01-shell-layout.md)

---

## 📐 Wireframe

```
┌─ Sidebar ─┬──────────────────────────────────────┐
│            │  Header: Project > Docs > Doc Title   │
│            ├──────────────────────────────────────┤
│            │                                      │
│            │  ┌─ Toolbar ────────────────────┐    │
│            │  │ B I U │ H1 H2 │ • │ </> │ 📎│    │
│            │  └──────────────────────────────┘    │
│            │                                      │
│            │  ┌─ Editor (max-w-3xl) ─────────┐   │
│            │  │                               │   │
│            │  │  # Document Title             │   │
│            │  │                               │   │
│            │  │  Content goes here...         │   │
│            │  │  Rich text editing with       │   │
│            │  │  markdown shortcuts.          │   │
│            │  │                               │   │
│            │  └───────────────────────────────┘   │
│            │                                      │
│            │  Auto-saved: 2 seconds ago            │
└────────────┴──────────────────────────────────────┘
```

---

## ✅ Chức năng

| Feature | Status | Mô tả |
|---------|--------|--------|
| Rich text editor (Tiptap/ProseMirror) | 🟡 Scale | Block-based editor |
| Toolbar (Bold, Italic, Headings) | 🟡 Scale | Formatting tools |
| Auto-save | 🟡 Scale | Debounced save (2s idle) |
| Markdown shortcuts | 🟡 Scale | `# `, `**`, `- ` |
| Code blocks | 🟡 Scale | Syntax highlight |
| Image upload | 🟡 Scale | Drag & drop |
| Version history | 🔴 Coming | Track changes |
| Real-time collaboration | 🔴 Coming | Multi-user editing |

---

## 📡 API Endpoints

| Method | Endpoint | Mô tả |
|--------|----------|--------|
| `GET` | `/documents/:id` | Load document content |
| `PATCH` | `/documents/:id` | Auto-save content |

---

*Last Updated: 2026-02-11*
