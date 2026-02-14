# Vector DB - ChromaDB Implementation Plan

**Status:** 🏗️ Phase 2  
**Tech:** ChromaDB v0.5+ (self-hosted, Docker)  
**Port:** `8001`  
**Embedding:** nomic-embed-text (Ollama) → fallback OpenAI

---

## 📐 Architecture Overview

```
┌─────────────────────────────────────────────────────────┐
│                    WorkflowHub Backend                   │
│                                                         │
│  ┌──────────────┐    ┌──────────────┐                   │
│  │  Documents    │    │  RAG Service  │                   │
│  │  Module       │───▶│  (Ingestor)  │                   │
│  │  (MySQL)      │    │              │                   │
│  └──────────────┘    └──────┬───────┘                   │
│                             │                           │
│                     ┌───────▼────────┐                  │
│                     │ Embedding Layer │                  │
│                     │ (Ollama/OpenAI) │                  │
│                     └───────┬────────┘                  │
│                             │                           │
└─────────────────────────────┼───────────────────────────┘
                              │ HTTP :8001
                    ┌─────────▼──────────┐
                    │     ChromaDB       │
                    │  (Docker Container) │
                    │                    │
                    │  Collection:       │
                    │  "workflowhub_docs" │
                    │                    │
                    │  Storage: /data    │
                    │  (Docker Volume)   │
                    └────────────────────┘
```

---

## 🐳 Docker Setup

### docker-compose.yml

```yaml
services:
  chromadb:
    image: chromadb/chroma:0.5.23
    container_name: workflowhub-chromadb
    ports:
      - "8001:8000"  # Host:Container
    volumes:
      - chroma_data:/chroma/chroma  # Persistent storage
    environment:
      - IS_PERSISTENT=TRUE
      - ANONYMIZED_TELEMETRY=FALSE
      - CHROMA_SERVER_AUTHN_PROVIDER=chromadb.auth.token_authn.TokenAuthenticationServerProvider
      - CHROMA_SERVER_AUTHN_CREDENTIALS=${CHROMA_AUTH_TOKEN}
      - CHROMA_SERVER_AUTHN_TOKEN_TRANSPORT_HEADER=Authorization
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8000/api/v2/heartbeat"]
      interval: 30s
      timeout: 10s
      retries: 3
    restart: unless-stopped

volumes:
  chroma_data:
    driver: local
```

### Environment Variables

```env
# .env
CHROMA_HOST=localhost
CHROMA_PORT=8001
CHROMA_AUTH_TOKEN=whub-chroma-secret-token-change-in-prod
CHROMA_COLLECTION_NAME=workflowhub_docs

# Embedding
EMBEDDING_PROVIDER=ollama          # ollama | openai
OLLAMA_HOST=http://localhost:11434
OLLAMA_EMBED_MODEL=nomic-embed-text
OPENAI_API_KEY=sk-...              # Fallback
OPENAI_EMBED_MODEL=text-embedding-3-small
```

---

## 📊 Collection Design

### Single Collection Strategy

**Collection:** `workflowhub_docs`

> **Rationale:** ChromaDB không hỗ trợ cross-collection queries. Dùng 1 collection + metadata filtering để isolate tenants → đơn giản, hiệu quả.

### Document Schema (per Chunk)

```typescript
interface ChromaDocument {
  // ChromaDB fields
  id: string;              // Format: "{document_id}__chunk_{index}"
  documents: string;       // Chunk text content
  embeddings: number[];    // Vector (768d nomic / 1536d OpenAI)
  
  // Metadata (filterable)
  metadatas: {
    project_id: string | null; // Scope narrowing
    document_id: string;       // Source reference
    document_title: string;    // For citation display
    chunk_index: number;       // Ordering within doc
    total_chunks: number;      // Total chunks in doc
    content_type: string;      // 'markdown' | 'rich_text'
    version: number;           // Doc version at embed time
    created_at: string;        // ISO 8601
  };
}
```

### ID Format

```
Format: {document_id}__chunk_{zero_padded_index}

Examples:
  abc123__chunk_000
  abc123__chunk_001
  abc123__chunk_042
```

**Why:** Cho phép upsert/delete toàn bộ chunks của 1 document dễ dàng bằng `where: { document_id: "abc123" }`.

---

## 🔪 Chunking Strategy

### Config

```typescript
/** @constant CHUNKING_CONFIG */
const CHUNKING_CONFIG = {
  /** Maximum characters per chunk */
  CHUNK_SIZE: 800,
  
  /** Overlap between chunks (giữ ngữ cảnh) */
  CHUNK_OVERLAP: 80, // 10% of CHUNK_SIZE
  
  /** Minimum chunk size (skip very small chunks) */
  MIN_CHUNK_SIZE: 50,
  
  /** Separators (ordered by priority) */
  SEPARATORS: [
    '\n## ',       // H2 headers (major sections)
    '\n### ',      // H3 headers
    '\n\n',        // Paragraph breaks
    '\n',          // Line breaks
    '. ',          // Sentences
    ' ',           // Words (last resort)
  ],
};
```

---

## 🧮 Embedding Layer

### Dual Provider Architecture

```typescript
/**
 * EmbeddingService
 * @description Hybrid embedding provider with automatic fallback
 */
class EmbeddingService {
  /**
   * Generate embeddings for text chunks
   * @param {string[]} texts - Array of text chunks
   * @returns {Promise<number[][]>} Array of embedding vectors
   */
  async embed(texts) {
    try {
      return await this.primaryProvider.embed(texts);
    } catch (error) {
      logger.warn('Primary embedding failed, falling back', { error });
      return await this.fallbackProvider.embed(texts);
    }
  }
}
```

---

## 🔒 Scope & Filtering

### Agent-Scoped Query Pattern

```typescript
/**
 * Query MUST include document filters based on Agent access
 */
const results = await collection.query({
  queryEmbeddings: [queryVector],
  where: {
    document_id: { $in: allowedDocIds }, // 🔒 REQUIRED - Agent knowledge scope
    project_id: projectId || undefined   // Optional scope
  },
  nResults: 10,
});
```

---

## 🔄 Sync Pipeline (MySQL ↔ ChromaDB)

### Sync Service

```typescript
/**
 * VectorSyncService
 * @description Handles MySQL → ChromaDB synchronization
 */
class VectorSyncService {
  /**
   * Ingest a document into ChromaDB
   * @param {string} documentId - Document UUID
   */
  async ingestDocument(documentId) {
    // 1. Fetch document from MySQL
    const doc = await documentRepo.findById(documentId);
    
    // ... logic ...
    
    // 6. Upsert to ChromaDB
    await this.collection.upsert({
      ids: chunks.map((_, i) => `${documentId}__chunk_${String(i).padStart(3, '0')}`),
      documents: chunks.map(c => c.text),
      embeddings: embeddings,
      metadatas: chunks.map((c, i) => ({
        project_id: doc.project_id || '',
        document_id: documentId,
        document_title: doc.title,
        chunk_index: i,
        total_chunks: chunks.length,
        content_type: doc.content_type,
        version: doc.version,
        created_at: new Date().toISOString(),
      })),
    });
    
    // 7. Update MySQL status
    await documentRepo.update(documentId, {
      embedding_status: 'completed',
      last_embedded_at: new Date(),
    });
  }
}
```

---

## 🔍 Query Interface

### RAG Query Flow

```typescript
/**
 * Query relevant document chunks
 * @param {string} query - User's question
 * @param {Object} [options] - Query options
 * @returns {Promise<RagResult[]>}
 */
async queryKnowledge(query, options = {}) {
  const {
    allowedDocIds = [], // MANDATORY: From agent_documents table
    projectId = null,
    topK = 5,
    minScore = 0.65,
  } = options;
  
  // 1. Embed query...
  
  // 2. Build filter
  const where = { document_id: { $in: allowedDocIds } };
  if (projectId) {
    where.project_id = projectId;
  }
  
  // 3. Query ChromaDB...
}
```

---

## 📊 Monitoring & Health

### Dashboard Metrics

```typescript
// GET /api/v1/analytics/vector
async getVectorStats() {
  const results = await collection.get({
    include: ['metadatas'],
  });
  
  const docIds = new Set(results.metadatas.map(m => m.document_id));
  
  return {
    total_chunks: results.ids.length,
    total_documents: docIds.size,
    avg_chunks_per_doc: results.ids.length / docIds.size,
    embedding_model: config.EMBEDDING_PROVIDER,
  };
}
```

---

## ⚡ Performance Optimization

### Batch Processing

```typescript
const BATCH_CONFIG = {
  /** Max chunks per embed call */
  EMBED_BATCH_SIZE: 32,
  
  /** Max chunks per ChromaDB upsert */
  UPSERT_BATCH_SIZE: 100,
  
  /** Concurrent document processing */
  CONCURRENCY: 3,
};
```

### Indexing Strategy

ChromaDB sử dụng HNSW (Hierarchical Navigable Small World) internally:

```typescript
// Collection creation with tuned HNSW params
const collection = await client.getOrCreateCollection({
  name: COLLECTION_NAME,
  metadata: {
    'hnsw:space': 'cosine',     // Distance metric
    'hnsw:construction_ef': 128, // Build quality (higher = better but slower)
    'hnsw:search_ef': 64,       // Search quality
    'hnsw:M': 16,               // Graph connectivity
  },
});
```

### Scaling Estimates

| Documents | Avg Chunks/Doc | Total Chunks | ChromaDB RAM | Disk |
|-----------|---------------|--------------|-------------|------|
| 100 | 10 | 1,000 | ~50 MB | ~20 MB |
| 1,000 | 10 | 10,000 | ~500 MB | ~200 MB |
| 10,000 | 10 | 100,000 | ~5 GB | ~2 GB |
| 100,000 | 10 | 1,000,000 | ~50 GB | ~20 GB |

> **MVP Target:** < 10,000 documents → ChromaDB single node đủ dùng.

---

## 🗂️ File Structure

```
apps/api/src/
├── services/
│   ├── vector/
│   │   ├── chroma.client.js         # ChromaDB connection
│   │   ├── embedding.service.js     # Ollama/OpenAI embeddings
│   │   ├── chunker.service.js       # Text chunking logic
│   │   ├── vector-sync.service.js   # MySQL ↔ Chroma sync
│   │   └── vector.constants.js      # Config & constants
│   │
│   └── rag/
│       ├── rag.service.js           # Query interface
│       └── rag.constants.js
│
├── jobs/
│   └── embedding.worker.js          # BullMQ worker
│
└── config/
    └── chroma.config.js             # Connection config
```

---

## 📚 Related Documents

- [01-rag-service.md](./01-rag-service.md) - RAG pipeline overview
- [02-ai-gateway.md](./02-ai-gateway.md) - AI orchestration (consumer)
- [../../database/tables/documents.md](../../database/tables/documents.md) - Source schema
- [../../database/redis-strategy.md](../../database/redis-strategy.md) - Redis caching

---

*Last Updated: 2026-02-13*
