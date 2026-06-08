# Resume Analyzer

## Goal

Build a system that evaluates resumes, extracts structured insights, and returns interview-focused feedback.

## Architecture

```text
[Resume Upload]
     |
     v
[FastAPI]
     |
     +--> [Parser]
     +--> [Prompt / Schema Layer]
     +--> [LLM]
     +--> [PostgreSQL Results]
```

## Folder Structure

```text
resume-analyzer/
  app/
    api/
    parsers/
    prompts/
    services/
    schemas/
  tests/
```

## Implementation Steps

1. Accept PDF or text resume input.
2. Extract or normalize plain text.
3. Send prompt with structured output schema.
4. Return scores, strengths, weaknesses, and suggested questions.
5. Log failures for malformed files or invalid model output.

## Interview Talking Points

- Why structured outputs are critical here
- How to validate model output before returning it
- How you would reduce inconsistent scoring

## Production Considerations

- Schema validation and retries
- PII handling
- Prompt versioning
- Benchmark examples for quality control

## Related Topics

- [structured-outputs.md](/c:/Users/ADMIN/Local%20Drive/applications/production-genai-engineer-roadmap/knowledge-base/07-Prompt-Engineering/structured-outputs.md)
- [prompt-testing-and-versioning.md](/c:/Users/ADMIN/Local%20Drive/applications/production-genai-engineer-roadmap/knowledge-base/07-Prompt-Engineering/prompt-testing-and-versioning.md)
