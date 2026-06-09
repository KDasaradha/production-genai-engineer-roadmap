# Fine-Tuned Domain Assistant

## Goal

Build a domain-specific assistant whose behavior or style is improved through fine-tuning rather than prompting alone.

## Architecture

```text
[User]
  |
  v
[FastAPI]
  |
  +--> [Fine-Tuned Model]
  +--> [Optional Retrieval Layer]
  +--> [Evaluation Pipeline]
```

## Folder Structure

```text
fine-tuned-domain-assistant/
  app/
    api/
    training/
    evaluation/
    prompts/
  tests/
```

## Implementation Steps

1. Pick a narrow domain and define desired behavior improvements.
2. Prepare training data and evaluation sets.
3. Fine-tune the model with LoRA/QLoRA or similar methods.
4. Compare tuned vs base model behavior.
5. Serve the model with evaluation-backed release criteria.

## Interview Talking Points

- Fine-tuning for behavior vs RAG for changing knowledge
- Dataset quality as the primary bottleneck
- Why evaluation matters before deployment

## Production Considerations

- Dataset versioning
- Overfitting risks
- Serving cost and latency
- Safety and regression evaluation

## Related Topics

- [fine-tuning-lora-qlora-peft.md](../11-LLM-Engineering/fine-tuning-lora-qlora-peft.md)
- [career-roadmap-and-study-plan.md](../14-Career/career-roadmap-and-study-plan.md)
