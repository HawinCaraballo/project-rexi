# TASK: Backend — Phase 4 — AI Document Bot Module

**Phase:** 4 (Advanced)  
**Module:** Documents / AI  
**Priority:** 🟢 Advanced  
**Estimate:** 4 days  
**Depends on:** PHASE_1_database_erd, PHASE_1_auth_module  

---

## Objective

Build a RAG-based AI chatbot that answers resident questions about complex rules and documents, using Azure OpenAI + Semantic Kernel + pgvector.

---

## Endpoints

### Document Management
| Method | Route | Role | Description |
|--------|-------|------|-------------|
| GET | `/api/v1/complexes/{cid}/documents` | Auth | List complex documents |
| POST | `/api/v1/complexes/{cid}/documents` | Admin | Upload document |
| DELETE | `/api/v1/documents/{id}` | Admin | Delete document |
| GET | `/api/v1/documents/{id}/download` | Auth | Download document |

### AI Chat
| Method | Route | Role | Description |
|--------|-------|------|-------------|
| POST | `/api/v1/complexes/{cid}/ai/chat` | Auth | Send message to bot |
| GET | `/api/v1/complexes/{cid}/ai/history` | Auth | Chat history (last 20 messages) |

---

## Document Upload & Indexing Flow

```
1. Admin uploads PDF/Word to POST /documents
2. File saved to Azure Blob Storage
3. DocumentIndexingJob triggered (Hangfire)
4. Job extracts text from file (PdfPig for PDF, DocumentFormat.OpenXml for .docx)
5. Text split into chunks (800 tokens, 100 token overlap)
6. Each chunk embedded via Azure OpenAI text-embedding-3-small
7. Embeddings stored in documents.document_chunks with pgvector
```

---

## Chat Endpoint Implementation

```csharp
// POST /api/v1/complexes/{cid}/ai/chat
// Body: { "message": "Can I have a pet in my apartment?" }
// Response: { "answer": "...", "sources": [{ "documentName": "...", "excerpt": "..." }] }
```

### RAG Pipeline

```csharp
public async Task<ChatResponseDto> ChatAsync(Guid complexId, string userMessage)
{
    // 1. Embed the user message
    var queryEmbedding = await _embeddingService.EmbedAsync(userMessage);
    
    // 2. Semantic search in pgvector
    var relevantChunks = await _uow.DocumentChunks
        .SearchSimilarAsync(complexId, queryEmbedding, topK: 5, threshold: 0.75);
    
    // 3. Build context
    var context = string.Join("\n\n", relevantChunks.Select(c => c.Content));
    
    // 4. Call Azure OpenAI
    var systemPrompt = $"""
        You are Rexi, a helpful assistant for the residential complex. 
        Answer ONLY based on the provided context documents.
        If the answer is not in the documents, say you don't have that information 
        and suggest contacting the administration.
        Context:
        {context}
        """;
    
    var response = await _openAiClient.GetChatCompletionsAsync(
        deploymentName: "gpt-4o-mini",
        messages: [
            new ChatMessage(ChatRole.System, systemPrompt),
            new ChatMessage(ChatRole.User, userMessage)
        ]
    );
    
    return new ChatResponseDto(
        Answer: response.Value.Choices[0].Message.Content,
        Sources: relevantChunks.Select(c => new SourceDto(c.DocumentName, c.Content[..200]))
    );
}
```

---

## pgvector Similarity Search

```csharp
// Repository method using raw SQL for pgvector cosine similarity:
public async Task<List<DocumentChunk>> SearchSimilarAsync(
    Guid complexId, float[] embedding, int topK, float threshold)
{
    var embeddingStr = $"[{string.Join(",", embedding)}]";
    return await _context.DocumentChunks
        .FromSqlRaw(@"
            SELECT dc.* FROM documents.document_chunks dc
            JOIN documents.complex_documents d ON d.id = dc.document_id
            WHERE d.complex_id = {0}
              AND 1 - (dc.embedding <=> {1}::vector) >= {2}
            ORDER BY dc.embedding <=> {1}::vector
            LIMIT {3}",
            complexId, embeddingStr, threshold, topK)
        .ToListAsync();
}
```

---

## Document Text Extraction

**PDF:** Use `PdfPig` library
```csharp
using UglyToad.PdfPig;
using var doc = PdfDocument.Open(fileBytes);
var text = string.Join("\n", doc.GetPages().Select(p => p.Text));
```

**DOCX:** Use `DocumentFormat.OpenXml`
```csharp
using DocumentFormat.OpenXml.Packaging;
using var doc = WordprocessingDocument.Open(stream, false);
var text = doc.MainDocumentPart.Document.Body.InnerText;
```

---

## Text Chunking Strategy

```csharp
// Chunk by paragraphs, max 800 tokens (~3200 chars), 100 token overlap
// Use tiktoken-sharp for token counting
public List<string> ChunkText(string text, int maxTokens = 800, int overlap = 100)
{
    // Split by double newlines (paragraphs)
    // Merge short paragraphs until max tokens reached
    // Add overlap from previous chunk
}
```

---

## Business Rules

- Bot only answers from the complex's own documents (scoped by complexId)
- Low confidence (no relevant chunks found) → polite escalation message
- Chat history stored per user per complex (last 30 days, 100 messages max)
- Document deletion triggers chunk cleanup
- Supports Spanish and English (model detects language automatically)

---

## Acceptance Criteria

- [ ] Document upload triggers indexing job
- [ ] Chunks stored with embeddings in pgvector
- [ ] Chat returns answer sourced from complex documents
- [ ] Sources included in response (document name + excerpt)
- [ ] Off-topic questions → polite "I don't have that information" response
- [ ] Deleted documents → chunks removed, no longer searchable
- [ ] Chat history stored and retrievable

---

## Unit Tests

- [ ] `TextChunker` — splits text correctly with overlap
- [ ] `TextChunker` — handles text shorter than chunk size
- [ ] `RagChatService` — returns fallback when no relevant chunks

---

## Integration Tests

- [ ] POST `/documents` → document indexed, chunks created
- [ ] POST `/ai/chat` with question about uploaded document → relevant answer
- [ ] POST `/ai/chat` with unrelated question → polite no-answer response
