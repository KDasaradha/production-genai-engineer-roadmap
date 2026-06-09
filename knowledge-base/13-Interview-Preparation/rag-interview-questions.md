# RAG Interview Questions

## Questions

1. What problem does RAG solve?
Answer: It grounds generation in retrieved external knowledge so the system can answer with fresher or domain-specific information.

2. Why does chunk size matter?
Answer: It affects context quality, retrieval precision, and context window efficiency.

3. Why can RAG still hallucinate?
Answer: Retrieval can fail, retrieved evidence can be weak or noisy, and the generation step can still misinterpret the context.

4. Why use reranking?
Answer: Reranking improves the final evidence set quality, especially when initial retrieval is broad or noisy.

5. When do you need a vector database?
Answer: When semantic retrieval, large-scale vector search, filtering, or operational vector indexing becomes a real requirement.

## Related Topics

- [rag-pipeline.md](../09-RAG/rag-pipeline.md)
- [advanced-retrieval.md](../09-RAG/advanced-retrieval.md)
