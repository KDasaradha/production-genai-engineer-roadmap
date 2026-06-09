# Agentic AI

Agentic AI systems let models use tools, maintain state, and complete multi-step workflows. Study this after LLM foundations, prompting, and RAG.

## Learning Order

| Order | Topic | Why It Comes Here | Output |
| --- | --- | --- | --- |
| 1 | [agent-fundamentals.md](agent-fundamentals.md) | Learn what an agent is and when agent behavior is justified | Agent decision checklist |
| 2 | [tool-calling.md](tool-calling.md) | Tools connect the model to APIs, databases, search, and actions | Safe tool-calling service |
| 3 | [state-memory-and-orchestration.md](state-memory-and-orchestration.md) | Multi-step workflows need state, memory, retries, and control flow | Stateful workflow design |
| 4 | [agent-frameworks.md](agent-frameworks.md) | Frameworks speed development but add abstraction and lock-in | Framework selection memo |

## What To Master

| Area | Why It Matters |
| --- | --- |
| Tool calling | Turns model intent into controlled actions |
| State | Tracks progress across steps |
| Memory | Stores useful context without polluting every prompt |
| Orchestration | Controls loops, retries, branching, and stop conditions |
| Safety | Prevents tool abuse, data leakage, and runaway loops |
| Evaluation | Measures task success, not just response quality |

## Common Trap

Do not use agents for simple single-step tasks. A normal prompt, function call, workflow, or RAG pipeline is often cheaper and safer.

## Interview Focus

| Question | Strong Answer Should Mention |
| --- | --- |
| What is an AI agent? | LLM-driven system that can plan, call tools, use state, and iterate toward a goal |
| When not to use agents? | Simple deterministic tasks, strict workflows, high-risk actions without control |
| How do you make agents safe? | Tool permissions, validation, guardrails, budgets, timeouts, audit logs |
| What is orchestration? | Managing steps, state, branching, retries, and termination |

## Project Connection

Use this folder with [AI Workflow Automation](../12-Projects/ai-workflow-automation.md), [Coding Assistant](../12-Projects/coding-assistant.md), and [Multi-Agent Research System](../12-Projects/multi-agent-research-system.md).
