# Multi-Provider AI Playground

## Goal

Build a comparison tool that runs the same prompt against multiple model providers and shows output, latency, and cost tradeoffs.

## Architecture

```text
[User Prompt]
    |
    v
[FastAPI / UI]
    |
    +--> [Provider Adapter: OpenAI]
    +--> [Provider Adapter: Anthropic]
    +--> [Provider Adapter: Gemini]
    +--> [Comparison Layer]
```

## Folder Structure

```text
multi-provider-ai-playground/
  app/
    api/
    providers/
    evaluation/
    schemas/
  tests/
```

## Implementation Steps

1. Create a provider-agnostic request interface.
2. Add adapters for multiple model providers.
3. Run the same prompt against each provider.
4. Capture latency, token usage, and output structure.
5. Display side-by-side comparisons.

## Interview Talking Points

- Why an abstraction layer matters
- How provider behavior differs despite similar prompts
- Why evaluation should separate quality, cost, and latency

## Production Considerations

- Timeout handling by provider
- Rate limits and retries
- Cost tracking
- Output normalization for comparisons

## Related Topics

- [advanced-prompting.md](../07-Prompt-Engineering/advanced-prompting.md)
- [prompt-testing-and-versioning.md](../07-Prompt-Engineering/prompt-testing-and-versioning.md)
