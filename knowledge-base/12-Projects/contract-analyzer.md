# Contract Analyzer

## Goal

Build an AI system that extracts clauses, obligations, and risk signals from contracts using structured outputs.

## Architecture

```text
[Contract Upload]
      |
      v
[FastAPI]
      |
      +--> [Parser / OCR]
      +--> [Prompt + Schema Layer]
      +--> [LLM]
      +--> [Risk Summary Builder]
```

## Folder Structure

```text
contract-analyzer/
  app/
    api/
    parsing/
    prompts/
    services/
    schemas/
  tests/
```

## Implementation Steps

1. Parse PDF or text contracts.
2. Define extraction schema for clauses and risks.
3. Prompt the model for structured extraction.
4. Validate the result and add fallback handling.
5. Return highlighted risks and summary output.

## Interview Talking Points

- Why structured outputs matter more than free text here
- How to reduce hallucinated legal claims
- Why this should assist review, not replace legal approval

## Production Considerations

- PII and sensitive document handling
- OCR quality issues
- Schema drift and prompt versioning
- Human review workflow

## Related Topics

- [structured-outputs.md](../07-Prompt-Engineering/structured-outputs.md)
- [prompt-security-and-guardrails.md](../07-Prompt-Engineering/prompt-security-and-guardrails.md)
