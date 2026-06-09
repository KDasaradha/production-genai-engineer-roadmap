# Coding Assistant

## Goal

Build an AI assistant that explains code, retrieves relevant files, and can perform constrained coding workflows.

## Architecture

```text
[Developer]
   |
   v
[FastAPI / IDE Integration]
   |
   +--> [Repo Indexer]
   +--> [Code Retriever]
   +--> [LLM]
   +--> [Tool Layer]
```

## Folder Structure

```text
coding-assistant/
  app/
    api/
    indexing/
    retrieval/
    tools/
    prompts/
  tests/
```

## Implementation Steps

1. Index repository files and metadata.
2. Retrieve relevant files for a user question.
3. Summarize code or propose edits with clear boundaries.
4. Add tool restrictions and audit logs.
5. Evaluate answer usefulness on real repo tasks.

## Interview Talking Points

- Why code retrieval matters more than generic prompting
- How to avoid unsafe or over-broad tool usage
- Why repository chunking differs from document chunking

## Production Considerations

- Access control to codebases
- Tool safety and approval flows
- Context-window management for large repos
- Hallucination reduction through code grounding

## Related Topics

- [multi-agent-research-system.md](multi-agent-research-system.md)
- [tool-calling.md](../10-Agentic-AI/tool-calling.md)
