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
*   **Trigger:** Khi `documents` được tạo hoặc cập nhật trong MySQL.
*   **Chunking Strategy:**
    *   Sử dụng `RecursiveCharacterTextSplitter`.
    *   `chunk_size`: 500 - 1000 characters.
    *   `chunk_overlap`: 10% (để giữ ngữ cảnh giữa các đoạn).
*   **Metadata:** Mỗi chunk phải đính kèm:
    *   `organization_id` (Bắt buộc - dùng để lọc).
    *   `document_id` (Để dẫn nguồn).
    *   `project_id` (Để thu hẹp phạm vi tìm kiếm).

### 2. Embedding Layer
*   **Model:** `text-embedding-3-small` (OpenAI) hoặc `all-MiniLM-L6-v2` (Local).
*   **Process:** Chuyển văn bản thành vector 1536 chiều (nếu dùng OpenAI).

### 3. Vector Storage (Vector DB)
*   **Tech:** ChromaDB (Docker-ready) hoặc Pinecone.
*   **Isolation:** 
    *   Sử dụng **Metadata Filtering** trên trường `organization_id`.
    *   *Mục tiêu:* Tuyệt đối không để AI của Org A tìm thấy tài liệu của Org B.

### 4. Retriever (Bộ truy xuất)
*   **Similarity Search:** Sử dụng Cosine Similarity để tìm các đoạn văn bản gần nhất với câu hỏi của User.
*   **Reranking (Optional):** Chỉnh sửa lại thứ tự các đoạn văn bản dựa trên độ phù hợp thực tế trước khi gửi cho AI.

---

## 🔒 Security & Tenant Isolation

**MANDATORY QUERY PATTERN:**
Mọi yêu cầu tìm kiếm phải kèm theo `organization_id`.

```typescript
// Ví dụ logic tìm kiếm
const results = await vectorDb.query({
  queryVector: userQueryEmbedding,
  filter: {
    organization_id: req.orgId // BẮT BUỘC
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
