# RAG

RAG, or Retrieval-Augmented Generation, grounds LLM responses in external knowledge. It is one of the most important production GenAI patterns.

## Learning Order

| Order | Topic | Why It Comes Here | Output |
| --- | --- | --- | --- |
| 1 | [embedding-types.md](embedding-types.md) | Different embedding models affect retrieval quality, cost, and latency | Embedding selection criteria |
| 2 | [chunking-strategies.md](chunking-strategies.md) | Chunking determines what can be retrieved | Chunking experiment |
| 3 | [retrieval-strategies.md](retrieval-strategies.md) | Search can be semantic, keyword, hybrid, filtered, or multi-step | Retrieval decision table |
| 4 | [vector-databases.md](vector-databases.md) | Vector DBs store and search embeddings at scale | Vector DB architecture |
| 5 | [rag-pipeline.md](rag-pipeline.md) | Connect ingestion, retrieval, generation, and citations | End-to-end RAG flow |
| 6 | [advanced-retrieval.md](advanced-retrieval.md) | Production systems need query rewriting, hybrid search, and filters | Better retrieval strategy |
| 7 | [reranking-and-context-compression.md](reranking-and-context-compression.md) | Retrieved results often need ranking and trimming before generation | Reranking pipeline |
| 8 | [graph-rag-and-agentic-rag.md](graph-rag-and-agentic-rag.md) | Advanced systems combine relationships, planning, and retrieval | Advanced RAG design |
| 9 | [rag-variants-and-production-roadmap.md](rag-variants-and-production-roadmap.md) | RAG variants are useful only when matched to real product needs | Variant decision framework |
| 10 | [rag-frameworks.md](rag-frameworks.md) | Frameworks help but introduce abstractions and tradeoffs | Build-vs-framework decision |

## What To Master

| Area | Why It Matters |
| --- | --- |
| Ingestion | Bad parsing creates bad retrieval |
| Chunking | Controls granularity and context quality |
| Embeddings | Affect semantic match quality |
| Retrieval | Determines what evidence the model sees |
| Reranking | Improves top results before generation |
| Citations | Build user trust and auditability |
| Evaluation | Reveals retrieval and answer quality failures |

## Common Trap

Do not blame the LLM first when RAG answers are bad. Most failures come from ingestion, chunking, retrieval, metadata filters, reranking, or prompt context.

## Interview Focus

| Question | Strong Answer Should Mention |
| --- | --- |
| What is RAG? | Retrieve relevant context, augment prompt, generate grounded answer |
| How do you improve RAG quality? | Chunking, hybrid search, metadata filters, reranking, evals, citations |
| Vector DB vs PostgreSQL? | Vector similarity vs relational queries, hybrid options, scale tradeoffs |
| What are RAG failure modes? | Bad chunks, wrong retrieval, stale data, missing citations, hallucinations |

## Project Connection

Build [Semantic Search Engine](../12-Projects/semantic-search-engine.md), then [Knowledge Assistant](../12-Projects/knowledge-assistant.md), then [Enterprise RAG Platform](../12-Projects/enterprise-rag-platform.md), then the [UK Council Planning Portal AI Platform](../12-Projects/uk-council-planning-portal-ai-platform.md) as a domain-specific production system.
