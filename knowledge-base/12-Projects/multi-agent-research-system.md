# Multi-Agent Research System

## Goal

Build a research workflow that plans subtasks, uses tools safely, stores intermediate state, and produces a final cited report.

## Architecture

```text
[User]
  |
  v
[FastAPI API]
  |
  v
[Agent Orchestrator]
  |
  +--> [Planner]
  +--> [Search Tool]
  +--> [Reader Tool]
  +--> [Memory Store]
  +--> [Evaluator / Loop Guard]
  +--> [Report Writer]
```

## Folder Structure

```text
multi-agent-research-system/
  app/
    agents/
    tools/
    orchestration/
    memory/
    api/
    schemas/
  tests/
  prompts/
  docs/
```

## Implementation Steps

1. Define agent state, tool schemas, and loop limits.
2. Build planner, search, and reader stages as explicit components.
3. Persist intermediate notes and citations in a memory store.
4. Add evaluator checks for duplication, loop control, and missing evidence.
5. Produce a final structured report with citations and traceability.
6. Add observability for tool failures, retries, and token/cost usage.

## Code References

- [agent-fundamentals.md](../10-Agentic-AI/agent-fundamentals.md)
- [tool-calling.md](../10-Agentic-AI/tool-calling.md)
- [state-memory-and-orchestration.md](../10-Agentic-AI/state-memory-and-orchestration.md)

## Interview Talking Points

- When to use an agent instead of a deterministic workflow
- How to prevent infinite loops and duplicate tool calls
- Why memory needs structure, not just appended chat history
- How to audit tool usage and citations

## Production Considerations

- Treat tools as failure-prone external dependencies
- Store tool outputs and reasoning traces for debugging
- Limit scope and authority of each tool
- Prefer explicit graph-style orchestration for testability

## Related Topics

- [backend-and-ai-scenarios.md](../05-System-Design/backend-and-ai-scenarios.md)
- [portfolio-projects.md](portfolio-projects.md)
