# Prompt Evaluation System

## Goal

Build a benchmark tool that compares prompt variants against expected outputs or evaluation criteria.

## Architecture

```text
[Test Cases]
    |
    v
[Evaluation Runner]
    |
    +--> [Prompt Versions]
    +--> [Model Provider]
    +--> [Scoring Layer]
    +--> [Results Store]
```

## Folder Structure

```text
prompt-evaluation-system/
  app/
    runners/
    prompts/
    scoring/
    reports/
  tests/
```

## Implementation Steps

1. Create test cases and expected evaluation criteria.
2. Store multiple prompt versions.
3. Run prompts against the same inputs.
4. Score structure, quality, latency, and cost.
5. Persist comparison reports.

## Interview Talking Points

- Why prompt changes need regression testing
- Deterministic vs subjective evaluation signals
- Why cost and latency belong in evaluation too

## Production Considerations

- Version control for prompts
- Reproducible test datasets
- Automated regression runs
- Human review for subjective tasks

## Related Topics

- [prompt-testing-and-versioning.md](/c:/Users/ADMIN/Local%20Drive/applications/production-genai-engineer-roadmap/knowledge-base/07-Prompt-Engineering/prompt-testing-and-versioning.md)
- [genai-interview-questions.md](/c:/Users/ADMIN/Local%20Drive/applications/production-genai-engineer-roadmap/knowledge-base/13-Interview-Preparation/genai-interview-questions.md)
