# Knowledge Assistant

## Goal

Build an assistant that answers questions from a curated knowledge base with source-aware retrieval.

## Architecture

```text
[User]
  |
  v
[FastAPI]
  |
  +--> [Retriever]
  +--> [Vector DB]
  +--> [Metadata / PostgreSQL]
  +--> [LLM]
```

## Folder Structure

```text
knowledge-assistant/
  app/
    api/
    ingestion/
    retrieval/
    generation/
    schemas/
  tests/
```

## Implementation Steps

1. Ingest documents and normalize metadata.
2. Chunk and embed documents.
3. Retrieve top matches for a query.
4. Build a grounded answer prompt using retrieved chunks.
5. Add citations and source snippets.

## Interview Talking Points

- Difference between a semantic search app and a full RAG assistant
- Why grounding matters for trust
- How permissions should affect retrieval

## Production Considerations

- Document update handling
- Source citation quality
- Query logging for failed retrieval
- Abuse prevention and rate limits

## Related Topics

- [rag-pipeline.md](../09-RAG/rag-pipeline.md)
- [enterprise-rag-platform.md](enterprise-rag-platform.md)
