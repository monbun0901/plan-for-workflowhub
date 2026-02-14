# RAG Service (Knowledge Engine)

**Status:** 🏗️ Phase 2 Design  
**Core Responsibility:** Cung cấp tri thức nội bộ cho AI Agent thông qua kỹ thuật Retrieval-Augmented Generation.

---

## 🛠️ Architecture

```
[ Documents (MySQL) ] ──▶ [ RAG Ingestor ] ──▶ [ Embedding Model ] ──▶ [ Vector DB ]
                                                                          │
[ User Query ] ─────────▶ [ RAG Retriever ] ◀─────────────────────────────┘
                                  │
                          [ Relevant Chunks ] ──▶ [ AI Gateway ]
```

---

## 📑 Detailed Components

### 1. Document Ingestor & Chunker
*   **Trigger:** Khi `documents` có `embedding_status = 'pending'`.
*   **Chunking Strategy:**
    *   Sử dụng `RecursiveCharacterTextSplitter`.
    *   `chunk_size`: 800 characters.
    *   `chunk_overlap`: 80 characters (10%).
*   **Metadata (Essential):** Mỗi chunk đính kèm:
    *   `document_id`: Để dẫn nguồn và xóa/sửa khi tài liệu thay đổi.
    *   `project_id`: (Optional) Để thu hẹp phạm vi tìm kiếm theo dự án.

### 2. Embedding Layer (Hybrid)
*   **Primary:** `nomic-embed-text` (Ollama - Local) -> Vector 768 chiều.
*   **Fallback:** `text-embedding-3-small` (OpenAI) -> Vector 1536 chiều.
*   *Lưu ý:* Cần chọn 1 loại Dimension cố định cho toàn bộ hệ thống.

### 3. Vector Storage (ChromaDB)
*   **Tech:** ChromaDB (Self-hosted trên Docker port 8001).
*   **Isolation:** Không cần `organization_id`. Cô lập tri thức ở tầng Query bằng `agent_documents`.

### 4. Retriever (Bộ truy xuất - Trí nhớ Agent)
*   **Agent Scoping:** Khi một Agent hỏi, Retriever CHỈ tìm kiếm trong các Document IDs được liệt kê trong bảng `agent_documents`.
*   **Logic:** `where: { document_id: { $in: [agent_allowed_doc_ids] } }`

---

## 🔒 Security & Scope

**AGENT-BASED FILTERING:**
Tuyệt đối không tìm kiếm mù quáng toàn server. AI Agent chỉ được biết những gì nó được "nạp" tri thức.

```typescript
// Logic tìm kiếm thực tế
const results = await vectorDb.query({
  queryVector: userQueryEmbedding,
  filter: {
    document_id: { $in: allowedDocIds } // Lấy từ bảng agent_documents
  },
  topK: 5
});
```

---

## 🚀 API Interface (Internal)

| Method | Endpoint | Description |
| :--- | :--- | :--- |
| `POST` | `/internal/rag/ingest` | Đẩy tài liệu mới vào Vector DB |
| `POST` | `/internal/rag/query` | Lấy các đoạn văn bản liên quan dựa trên query |
| `DELETE`| `/internal/rag/docs/:id` | Xóa tài liệu khỏi bộ nhớ Vector |

---

*Last Updated: 2026-02-11*
