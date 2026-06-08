# Chunking Strategies

## 1. Problem Statement

Chunking solves the problem of splitting large documents into smaller pieces that can be embedded, retrieved, cited, and passed into an LLM context window.

LLMs and embedding models cannot always process entire documents efficiently. Even when they can, sending full documents is expensive, slow, noisy, and often unnecessary. Chunking decides the unit of knowledge your retrieval system can find.

Without good chunking:

- retrieval returns incomplete context
- answers miss important details
- citations point to vague or wrong sections
- chunks are too small to be meaningful
- chunks are too large and noisy
- RAG quality becomes unstable

Real-world analogy: a textbook index points to specific pages or sections, not the entire book. Chunking creates those searchable sections for AI.

## 2. Beginner Explanation

| Item | Notes |
| --- | --- |
| Definition | Chunking is the process of splitting documents into smaller text units for embedding and retrieval. |
| Key terminology | chunk, overlap, token count, semantic boundary, parent-child retrieval, metadata, citation |
| Simple explanation | Break big documents into useful pieces so search can find the right part. |
| Mental model | Chunks are the paragraphs or sections your AI system can retrieve. |
| Easy example | Split a policy document by headings and paragraphs. |
| Use When | Documents are too large or broad to search and send as a whole. |
| Avoid When | Each record is already small and self-contained. |
| Advantages | Improves retrieval focus, context efficiency, and citations. |
| Tradeoffs | Bad chunking can lose context or add noise. |
| Limitations | No universal best chunk size exists. |
| Production Example | Legal RAG uses section-aware chunks with page numbers and clause metadata. |
| Interview Answer | Chunking controls the retrieval unit in RAG, and chunk size directly affects recall, precision, context quality, and citation usefulness. |

## 3. Intermediate Explanation

Common chunking strategies:

| Strategy | How It Works | Best For | Risk |
| --- | --- | --- | --- |
| Fixed-size | Split every N characters or tokens | quick baseline | may cut meaning |
| Overlapping | Fixed chunks with repeated overlap | context crossing boundaries | duplicated context |
| Recursive | Split by hierarchy: headings, paragraphs, sentences | docs and articles | more complex |
| Semantic | Split where meaning changes | high-quality docs | requires more processing |
| Parent-child | retrieve small child chunks, return larger parent sections | detailed retrieval with context | more storage logic |
| Sliding window | move fixed window through text | transcripts, logs | many chunks |
| Structure-aware | split by Markdown, HTML, PDF sections, code functions | structured docs and code | parser quality matters |

Important metadata:

- document ID
- chunk ID
- chunk index
- page number
- section heading
- source URI
- tenant ID
- permissions
- token count
- chunking strategy version

Data flow:

```text
Document -> parser -> chunker -> metadata enrichment -> embeddings -> vector DB
```

## 4. Advanced Explanation

Chunking should be evaluated by retrieval quality, not only by chunk size.

Optimization techniques:

- Split on semantic boundaries when possible.
- Add overlap for concepts that cross boundaries.
- Keep chunks small enough for precise retrieval.
- Keep chunks large enough to preserve meaning.
- Store parent context for citations and answer generation.
- Preserve document structure in metadata.
- Use different chunking for code, tables, PDFs, and chat logs.
- Version chunking strategy so reprocessing is possible.

Performance considerations:

- Smaller chunks increase index size.
- Larger chunks increase prompt token usage.
- Overlap increases storage and embedding cost.
- Semantic chunking can slow ingestion.
- Parent-child retrieval adds lookup complexity.

Scaling considerations:

- Chunk asynchronously during ingestion.
- Store chunking version.
- Re-chunk and re-embed when strategy changes.
- Monitor average chunks per document.
- Track retrieval quality by document type.

Production challenges:

- scanned PDFs with bad OCR
- tables split incorrectly
- headings lost during extraction
- code blocks split mid-function
- citations missing page numbers
- duplicate chunks from overlap

## 5. Internal Working

```text
Raw file
  |
  v
Text extraction
  |
  v
Structure detection
  |
  v
Chunk splitting
  |
  v
Overlap or parent-child mapping
  |
  v
Metadata assignment
  |
  v
Embedding generation
  |
  v
Storage in vector DB and metadata DB
```

Detailed lifecycle:

1. User uploads a document.
2. System extracts text and structure.
3. Chunker chooses boundaries.
4. Metadata is attached.
5. Chunks are embedded.
6. Chunks are stored and indexed.
7. Query retrieves relevant chunks.
8. Context builder sends selected chunks to the LLM.
9. Answer includes citations back to chunk metadata.

## 6. When To Use

Use chunking for:

- PDFs
- Markdown docs
- web pages
- contracts
- manuals
- support articles
- transcripts
- code repositories
- long chat histories

Ideal use cases:

- RAG
- semantic search
- document Q&A
- citations
- long-document summarization
- compliance review

## 7. When NOT To Use

Avoid chunking when:

- records are already small and complete
- exact structured database queries are better
- splitting would remove required context
- the data is already normalized into meaningful rows

Better alternatives:

- row-level retrieval for structured data
- SQL queries
- section-level indexing only
- full-document summarization before retrieval

## 8. Advantages

- Enables retrieval over long documents.
- Keeps prompts smaller.
- Improves search precision.
- Supports citations.
- Lets you update parts of documents.
- Helps control context window usage.

## 9. Tradeoffs

| Tradeoff | Explanation |
| --- | --- |
| Small chunks vs context | Small chunks are precise but may miss surrounding meaning. |
| Large chunks vs noise | Large chunks preserve context but add irrelevant tokens. |
| Overlap vs cost | Overlap preserves continuity but increases storage and embedding cost. |
| Simple vs accurate | Fixed-size chunking is easy but structure-aware chunking is often better. |

## 10. Limitations

- No single chunk size fits all documents.
- OCR errors reduce chunk quality.
- Tables and diagrams are difficult.
- Overlap can create duplicate retrieval.
- Chunking cannot recover information lost during extraction.
- Evaluation is required to know whether chunking works.

## 11. Real-World Examples

Startup example: a documentation assistant chunks Markdown docs by heading.

Enterprise example: a legal assistant chunks contracts by clauses and stores page numbers.

FAANG-style example: a knowledge platform uses document-type-specific chunkers and evaluates retrieval quality per corpus.

Production system: a RAG platform stores child chunks for precise search and parent sections for final context.

## 12. Architecture Diagram

```text
[PDF / Docs / HTML]
        |
        v
[Text + Structure Extractor]
        |
        v
[Chunking Strategy]
        |
        +--> [Child Chunks] -> [Embeddings] -> [Vector DB]
        |
        +--> [Parent Sections] -> [PostgreSQL Metadata]
```

Parent-child retrieval:

```text
Query -> retrieve child chunk -> fetch parent section -> send parent context to LLM
```

## 13. Python Implementation

Fixed-size chunking:

```python
def fixed_chunks(text: str, size: int) -> list[str]:
    return [text[index:index + size] for index in range(0, len(text), size)]
```

Overlapping chunking:

```python
def overlapping_chunks(text: str, size: int, overlap: int) -> list[str]:
    if overlap >= size:
        raise ValueError("overlap must be smaller than size")

    chunks: list[str] = []
    index = 0
    while index < len(text):
        chunks.append(text[index:index + size])
        index += size - overlap
    return chunks
```

Paragraph chunking:

```python
def paragraph_chunks(text: str, max_chars: int) -> list[str]:
    paragraphs = [item.strip() for item in text.split("\n\n") if item.strip()]
    chunks: list[str] = []
    current = ""

    for paragraph in paragraphs:
        if len(current) + len(paragraph) + 2 > max_chars and current:
            chunks.append(current)
            current = paragraph
        else:
            current = f"{current}\n\n{paragraph}".strip()

    if current:
        chunks.append(current)

    return chunks
```

Chunk record:

```python
from dataclasses import dataclass

@dataclass
class Chunk:
    chunk_id: str
    document_id: str
    text: str
    chunk_index: int
    metadata: dict[str, str]
```

## 14. FastAPI Implementation

```python
from fastapi import FastAPI
from pydantic import BaseModel, Field

app = FastAPI()

class ChunkPreviewRequest(BaseModel):
    document_text: str = Field(min_length=1)
    strategy: str = "paragraph"
    max_chars: int = Field(default=800, ge=100, le=5000)

class ChunkPreviewResponse(BaseModel):
    strategy: str
    chunk_count: int
    sample_chunks: list[str]

@app.post("/documents/chunk-preview", response_model=ChunkPreviewResponse)
async def preview_chunks(request: ChunkPreviewRequest) -> ChunkPreviewResponse:
    if request.strategy == "fixed":
        chunks = fixed_chunks(request.document_text, request.max_chars)
    else:
        chunks = paragraph_chunks(request.document_text, request.max_chars)

    return ChunkPreviewResponse(
        strategy=request.strategy,
        chunk_count=len(chunks),
        sample_chunks=chunks[:3],
    )
```

Production-ready structure:

```text
app/
  api/routes/documents.py
  services/text_extractor.py
  services/chunking_service.py
  services/embedding_worker.py
  repositories/document_repository.py
  repositories/chunk_repository.py
```

## 15. Database Integration

PostgreSQL tables:

```text
documents(id, tenant_id, title, source_uri, checksum, status, created_at)
document_sections(id, document_id, heading, start_page, end_page)
chunks(id, document_id, section_id, chunk_index, text, token_count, strategy_version)
```

Vector DB fields:

- chunk ID
- embedding vector
- document ID
- tenant ID
- page number
- section heading
- strategy version
- permission metadata

Redis use:

- track ingestion job progress
- deduplicate document processing by checksum
- cache chunk preview results

## 16. Production Considerations

- Preserve source and page metadata for citations.
- Version chunking strategy.
- Reprocess documents when chunking changes.
- Use async workers for large ingestion.
- Monitor chunk count per document.
- Detect empty or low-quality chunks.
- Handle OCR and table extraction carefully.
- Use document-type-specific chunkers.
- Evaluate chunking with retrieval test cases.

## 17. Common Mistakes

| Level | Mistake | How To Avoid |
| --- | --- | --- |
| Beginner | Splitting only by character count | Prefer headings, paragraphs, or semantic boundaries |
| Beginner | No metadata | Store document, page, section, and source |
| Intermediate | No overlap where needed | Add controlled overlap |
| Intermediate | Chunks too large | Tune chunk size with retrieval evaluation |
| Production | No chunking version | Store strategy version and support reprocessing |
| Production | Ignoring tables and code blocks | Use structure-aware parsers |

## 18. Interview Preparation

| Level | Question | Model Answer |
| --- | --- | --- |
| Basic | What is chunking? | Splitting documents into smaller retrievable units for embedding and RAG. |
| Basic | Why is chunking important? | It controls what context retrieval can return to the LLM. |
| Intermediate | What is overlap? | Repeating part of adjacent chunks to preserve boundary context. |
| Advanced | How do you choose chunk size? | Evaluate retrieval quality, context budget, document structure, citation needs, and token cost. |
| Scenario | RAG answers miss important surrounding context. | Increase chunk size, add overlap, use parent-child retrieval, or improve structure-aware splitting. |

## 19. System Design Discussion

Chunking is not preprocessing trivia. It defines the knowledge unit of your RAG system. Bad chunking causes retrieval failures that look like model failures.

Design decisions:

- fixed vs recursive vs semantic chunking
- overlap amount
- parent-child retrieval
- metadata fields
- chunking version
- document-type-specific chunkers
- reprocessing strategy

## 20. Hands-On Assignment

- Easy: Split a text file by paragraph.
- Medium: Add overlap and chunk metadata.
- Hard: Build a chunk preview endpoint and compare fixed vs paragraph chunking.

## 21. Mini Project

Build a Document Chunk Preview Tool.

Requirements:

- Accept raw text.
- Support fixed and paragraph chunking.
- Show chunk count.
- Show first 3 chunks.
- Include approximate token count.
- Include chunk metadata.

Folder structure:

```text
chunk-preview/
  app/
    main.py
    chunker.py
    schemas.py
  tests/
    test_chunker.py
```

## 22. Production-Level Project

Build a Versioned Document Ingestion Pipeline.

Real-world problem:

- A company needs reliable document ingestion for RAG with citations and reprocessing.

Architecture:

```text
[Upload] -> [Object Storage]
          -> [Text Extraction Worker]
          -> [Structure-Aware Chunker]
          -> [Embedding Worker]
          -> [Vector DB]
          -> [PostgreSQL Metadata]
```

Tech stack:

- FastAPI
- PostgreSQL
- Redis queue/state
- Vector database
- Object storage
- Background workers

Scaling strategy:

- Process documents asynchronously.
- Store checksums to avoid duplicates.
- Version chunking and embeddings.
- Reprocess in batches.
- Monitor failed ingestion jobs.

## Quiz

1. What is chunking?
2. Why is chunking important for RAG?
3. What happens if chunks are too small?
4. What happens if chunks are too large?
5. What is overlap?
6. What is recursive chunking?
7. What is semantic chunking?
8. What is parent-child retrieval?
9. Why store page and section metadata?
10. Why version chunking strategy?

## Knowledge Check

You should be able to design chunking strategies for different document types and explain how chunking affects retrieval, citations, cost, and answer quality.

Are you ready for the next section?
