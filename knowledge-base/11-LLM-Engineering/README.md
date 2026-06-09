# LLM Engineering

LLM engineering covers model customization, local serving, compression, and deployment tradeoffs.

## Learning Order

| Order | Topic | Why It Comes Here | Output |
| --- | --- | --- | --- |
| 1 | [local-models-ollama-hugging-face-vllm.md](local-models-ollama-hugging-face-vllm.md) | Learn when and how to run models outside hosted APIs | Local model serving plan |
| 2 | [fine-tuning-lora-qlora-peft.md](fine-tuning-lora-qlora-peft.md) | Fine-tuning is useful only after you understand data, prompts, and RAG | Fine-tuning decision framework |
| 3 | [quantization-and-distillation.md](quantization-and-distillation.md) | Compression affects cost, latency, memory, and quality | Model optimization tradeoff table |

## What To Master

| Area | Why It Matters |
| --- | --- |
| Local serving | Gives control but increases operations burden |
| Fine-tuning | Changes model behavior but requires data quality and evaluation |
| LoRA/QLoRA | Reduces training cost for adaptation |
| Quantization | Reduces memory and cost, may reduce quality |
| Distillation | Trains smaller models to imitate larger ones |
| Evaluation | Prevents optimization from silently hurting quality |

## Common Trap

Do not fine-tune because "the model is wrong" before checking prompt quality, retrieval quality, data freshness, and evaluation.

## Interview Focus

| Question | Strong Answer Should Mention |
| --- | --- |
| Fine-tuning vs RAG? | Fine-tuning changes behavior/style; RAG injects external knowledge |
| Why quantize? | Lower memory, lower cost, faster inference, quality tradeoff |
| When use local models? | Privacy, control, latency, cost, offline needs, ops tradeoffs |
| What is LoRA? | Parameter-efficient adaptation using low-rank updates |

## Project Connection

Use this folder with [Offline Local AI System](../12-Projects/offline-local-ai-system.md), [Fine-Tuned Domain Assistant](../12-Projects/fine-tuned-domain-assistant.md), and [Production AI Platform](../12-Projects/production-ai-platform.md).
