# Generative AI

This folder teaches the LLM foundations required before prompt engineering, RAG, agents, fine-tuning, and system design.

## Learning Order

| Order | Topic | Why It Comes Here | Output |
| --- | --- | --- | --- |
| 1 | [tokens-and-tokenization.md](tokens-and-tokenization.md) | Tokens affect cost, limits, latency, and output length | Token budget mental model |
| 2 | [transformer-basics.md](transformer-basics.md) | Transformers explain attention, context, and next-token prediction | High-level architecture understanding |
| 3 | [context-windows.md](context-windows.md) | Context limits shape prompt, RAG, and memory design | Context management strategy |
| 4 | [temperature-and-top-p.md](temperature-and-top-p.md) | Generation settings control randomness and reliability | Parameter tuning guide |
| 5 | [hallucinations.md](hallucinations.md) | Production AI systems must detect and reduce unsupported answers | Hallucination mitigation checklist |
| 6 | [embeddings.md](embeddings.md) | Embeddings power semantic search and RAG | Vector representation mental model |
| 7 | [cosine-similarity.md](cosine-similarity.md) | Similarity scoring explains retrieval behavior | Similarity search demo |

## What To Master

| Area | Why It Matters |
| --- | --- |
| Tokens | Pricing, limits, latency, truncation |
| Context | Determines what the model can use |
| Parameters | Affect creativity, determinism, and risk |
| Hallucinations | Define when grounding and verification are needed |
| Embeddings | Convert meaning into vectors for retrieval |
| Similarity | Explains why search results match or fail |

## Common Trap

Do not say LLMs "understand" text like humans. In interviews, describe prediction, representations, context, probability, and limitations clearly.

## Interview Focus

| Question | Strong Answer Should Mention |
| --- | --- |
| What is a token? | Subword/text unit, cost, context limit, generation |
| What is an embedding? | Dense vector representation of semantic meaning |
| Why hallucinations happen? | Probabilistic generation, missing grounding, ambiguous context |
| Temperature vs top-p? | Sampling controls, creativity vs consistency tradeoff |

## Project Connection

Use these foundations in [Semantic Search Engine](../12-Projects/semantic-search-engine.md), [Knowledge Assistant](../12-Projects/knowledge-assistant.md), and [Enterprise RAG Platform](../12-Projects/enterprise-rag-platform.md).
