# Documents Module

**Version:** v1  
**Date:** 2026-02-11  
**Skills:** `backend-architect`, `nodejs-best-practices`

---

## 🎯 Overview

Documents module quản lý knowledge base documents + RAG integration cho AI.

**Status:** 📝 To Be Implemented  
**Pattern:** Follow [04-projects-module.md](04-projects-module.md) + Vector DB integration

---

## 📋 Implementation Guide

### Special Requirements

**Document module khác Projects vì cần:**
- File upload handling
- Vector embeddings (Chroma)
- Search functionality
- RAG pipeline integration

### Key Services

```typescript
// apps/api/src/modules/documents/services/document.service.ts
export class DocumentService {
  constructor(
    private readonly documentRepo: DocumentRepository,
    private readonly vectorService: VectorService
  ) {}
  
  /**
   * Create document AND generate embeddings
   */
  async createDocument(organizationId: string, dto: CreateDocumentDto) {
    // 1. Save document to DB
    const document = await this.documentRepo.create({
      ...dto,
      organization_id: organizationId,
    });
    
    // 2. Generate embeddings
    await this.vectorService.createEmbedding({
      documentId: document.id,
      content: dto.content,
      organizationId, // CRITICAL: Tenant isolation in vector DB
    });
    
    return document;
  }
}
```

### Vector Store Integration

```typescript
// apps/api/src/modules/documents/services/vector.service.ts
import { ChromaClient } from 'chromadb';

export class VectorService {
  private client: ChromaClient;
  
  async createEmbedding(data: {
    documentId: string;
    content: string;
    organizationId: string;
  }) {
    const collection = await this.client.getOrCreateCollection({
      name: `org_${data.organizationId}` // Separate collection per org
    });
    
    await collection.add({
      ids: [data.documentId],
      documents: [data.content],
      metadatas: [{
        organization_id: data.organizationId,
        document_id: data.documentId,
      }],
    });
  }
  
  async search(organizationId: string, query: string, limit: number = 5) {
    const collection = await this.client.getCollection({
      name: `org_${organizationId}`
    });
    
    return collection.query({
      queryTexts: [query],
      nResults: limit,
    });
  }
}
```

---

## 🔒 Security Considerations

- ✅ **Tenant isolation in vector DB** - Separate collections per org
- ✅ **File upload validation** - File type, size limits
- ✅ **Content sanitization** - XSS prevention
- ✅ **Access control** - Document permissions

---

## 📚 Related Documents

- [04-projects-module.md](04-projects-module.md) - Base pattern
- [09-chat-module.md](09-chat-module.md) - RAG usage
- [../database/v1-database-security.md](../database/v1-database-security.md) - Vector DB security

---

*Status: Placeholder - Complex module requiring Vector DB integration*
