# Prompt Engineering

Prompt engineering is the control layer for model behavior. Study it after LLM foundations and before RAG and agents.

## Learning Order

| Order | Topic | Why It Comes Here | Output |
| --- | --- | --- | --- |
| 1 | [prompt-types.md](prompt-types.md) | Learn instruction, few-shot, role, constraint, and reasoning prompt patterns | Prompt pattern library |
| 2 | [advanced-prompting.md](advanced-prompting.md) | Improve reliability through decomposition, examples, and constraints | Strong task prompts |
| 3 | [structured-outputs.md](structured-outputs.md) | Production apps need predictable JSON, schemas, and validation | Schema-driven extraction API |
| 4 | [prompt-testing-and-versioning.md](prompt-testing-and-versioning.md) | Prompts need regression tests like code | Prompt evaluation workflow |
| 5 | [prompt-security-and-guardrails.md](prompt-security-and-guardrails.md) | Prompt injection and unsafe tool use are production risks | Guardrail checklist |

## What To Master

| Area | Why It Matters |
| --- | --- |
| Clear instructions | Reduce ambiguity |
| Examples | Teach the model output shape and edge cases |
| Constraints | Limit unwanted behavior |
| Structured outputs | Make AI responses usable by backend systems |
| Testing | Prevent prompt changes from silently breaking behavior |
| Security | Defend against prompt injection and data leakage |

## Common Trap

Do not rely on a clever prompt for tasks that need validation, retrieval, tools, permissions, or deterministic business rules.

## Interview Focus

| Question | Strong Answer Should Mention |
| --- | --- |
| What makes a good prompt? | Task, context, constraints, examples, output format, evaluation |
| What is structured output? | Schema-constrained response, validation, retries, safer integration |
| How do you test prompts? | Golden datasets, automated checks, regression testing, versioning |
| What is prompt injection? | Malicious instructions in input, data exfiltration, tool misuse, guardrails |

## Project Connection

Use this folder with [Prompt Evaluation System](../12-Projects/prompt-evaluation-system.md), [Resume Analyzer](../12-Projects/resume-analyzer.md), and [AI Security Gateway](../12-Projects/ai-security-gateway.md).
