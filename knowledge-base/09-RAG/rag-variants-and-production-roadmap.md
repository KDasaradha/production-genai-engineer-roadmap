# RAG Variants and Production Roadmap

This page consolidates the useful RAG variant notes from `archive/duplicate-sources/AI-RAG-Reasearch-notes/rag-chatbots/`.

## Core Recommendation

For job readiness, do not try to master every RAG variant equally. Most production systems are built from a small set of practical patterns:

| Priority | Variant | Learn Depth | Build Depth | Why |
| --- | --- | --- | --- | --- |
| 1 | Hybrid RAG | Deep | Must build | Combines keyword and semantic search for better recall |
| 2 | Advanced RAG | Deep | Must build | Adds rewriting, filters, reranking, citations, and evaluation |
| 3 | Agentic RAG | Deep | Build once | Useful when retrieval is part of a multi-step task |
| 4 | Semantic RAG | Medium | Build early | Foundation for vector search and embeddings |
| 5 | Graph RAG | Conceptual first | Optional | Useful for relationship-heavy domains |
| 6 | Multimodal RAG | Conceptual first | Optional | Useful for images, PDFs, OCR, and mixed media |
| 7 | Self-RAG | Conceptual | Optional | Model critiques whether retrieval or answer quality is enough |
| 8 | CRAG | Conceptual | Optional | Corrective retrieval when initial results are weak |
| 9 | Adaptive RAG | Conceptual | Optional | Chooses retrieval strategy based on query type |
| 10 | Naive RAG | Basic | Mini project only | Good for learning, weak for production |

## Mental Model

```text
Naive RAG
  -> Semantic RAG
  -> Hybrid RAG
  -> Advanced RAG
  -> Agentic or Graph RAG only when the use case requires it
```

## Variant Summary

| Variant | Definition | Use When | Avoid When | Interview Answer |
| --- | --- | --- | --- | --- |
| Naive RAG | Retrieve top-k chunks and place them in the prompt | Learning the basic flow | Production accuracy matters | Naive RAG is the simplest retrieve-then-generate pattern, but it often fails on recall, ranking, and citations. |
| Semantic RAG | Uses embeddings and vector similarity | Meaning-based search is enough | Exact terms, codes, or names matter | Semantic RAG finds conceptually similar text, but can miss exact keyword matches. |
| Hybrid RAG | Combines dense vector search with keyword/BM25 search | Enterprise documents, legal, policy, support | Very small datasets where simple search works | Hybrid RAG improves recall by combining semantic meaning and exact term matching. |
| Advanced RAG | Adds query rewriting, metadata filters, reranking, compression, citations, evals | Production RAG systems | Simple FAQ with stable answers | Advanced RAG is usually what companies mean by production RAG. |
| Agentic RAG | An agent decides when and how to retrieve, call tools, or continue | Multi-step workflows | Single question-answer retrieval | Agentic RAG is useful when retrieval is one action in a larger task plan. |
| Graph RAG | Uses entities and relationships to retrieve connected context | Relationship-heavy domains | Plain document Q&A | Graph RAG helps when relationships matter more than isolated chunks. |
| Multimodal RAG | Retrieves from text, images, OCR, tables, or PDFs | Visual or document-heavy products | Text-only knowledge base | Multimodal RAG grounds answers across multiple media types. |
| Self-RAG | The system evaluates whether retrieval and answer quality are sufficient | High-risk answers needing self-checks | Latency-sensitive simple apps | Self-RAG adds reflection, but increases complexity and latency. |
| CRAG | Corrective RAG retries or changes retrieval when evidence is weak | Retrieval quality varies | Retrieval is already reliable | CRAG improves bad retrieval paths by detecting weak evidence and correcting the strategy. |
| Adaptive RAG | Routes queries to different retrieval strategies | Mixed query types | One known query pattern | Adaptive RAG chooses the retrieval path based on query intent. |

## Production Learning Path

| Phase | Build | Skills Proven |
| --- | --- | --- |
| 1 | Basic document Q&A | Ingestion, chunking, embeddings, vector search |
| 2 | Semantic search API | Embedding model choice, similarity, metadata |
| 3 | Hybrid search API | BM25, vector search, reciprocal rank fusion |
| 4 | Advanced RAG assistant | Query rewriting, filters, reranking, citations |
| 5 | Evaluated RAG system | Retrieval metrics, answer quality, regression tests |
| 6 | Agentic RAG workflow | Tool calling, planning, state, task completion |
| 7 | Graph or multimodal extension | Specialized advanced retrieval |

## Common Mistakes

| Mistake | Why It Hurts | Better Approach |
| --- | --- | --- |
| Learning every variant before building | Creates theory overload | Build semantic, hybrid, and advanced RAG first |
| Using Graph RAG too early | Adds data modeling complexity | Start with metadata filtering and hybrid retrieval |
| Using agents for every RAG app | Adds cost, latency, and unpredictability | Use deterministic retrieval first |
| Ignoring evaluation | You cannot prove improvement | Track recall, precision, faithfulness, citation quality |
| Treating reranking as optional in production | Top-k vector results are often noisy | Add reranking for important workflows |

## Interview Answer

The most practical production path is semantic RAG first, then hybrid RAG, then advanced RAG with metadata filters, query rewriting, reranking, citations, and evaluation. Agentic RAG, Graph RAG, Self-RAG, CRAG, adaptive RAG, and multimodal RAG are useful extensions, but they should be introduced only when the product requirement justifies their extra complexity.

Are you ready for the next section?
