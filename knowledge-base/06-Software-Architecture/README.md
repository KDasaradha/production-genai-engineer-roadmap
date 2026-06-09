# Software Architecture

This folder covers the production layer around AI applications: backend patterns, UX, deployment, observability, CI/CD, cloud architecture, and cost.

## Learning Order

| Order | Topic | Why It Comes Here | Output |
| --- | --- | --- | --- |
| 1 | [ai-backend-patterns.md](ai-backend-patterns.md) | Establish service, queue, worker, and provider patterns | Backend architecture map |
| 2 | [ai-frontend-and-product-ux.md](ai-frontend-and-product-ux.md) | AI products need streaming, feedback, citations, and user trust | Product UX checklist |
| 3 | [docker-logging-api-scaling.md](docker-logging-api-scaling.md) | Basic deployment, logging, and scaling connect code to operations | Deployable API baseline |
| 4 | [observability-and-monitoring.md](observability-and-monitoring.md) | Production failures need traces, logs, metrics, and alerts | Monitoring plan |
| 5 | [docker-and-kubernetes.md](docker-and-kubernetes.md) | Containers and orchestration support repeatable deployment | Container and scaling strategy |
| 6 | [cicd-for-ai-systems.md](cicd-for-ai-systems.md) | AI systems need automated testing and deployment gates | CI/CD pipeline design |
| 7 | [cloud-ai-architecture.md](cloud-ai-architecture.md) | Cloud design affects latency, security, reliability, and cost | Cloud architecture diagram |
| 8 | [gpu-deployment-and-cost-optimization.md](gpu-deployment-and-cost-optimization.md) | Model serving can become the biggest cost center | Cost optimization plan |

## What To Master

| Area | Why It Matters |
| --- | --- |
| Service boundaries | Keeps APIs, workers, model calls, and storage maintainable |
| Observability | Makes model, retrieval, and backend failures diagnosable |
| Security | Protects data, prompts, tools, and provider keys |
| Deployment | Moves from notebook/demo to repeatable production |
| Cost controls | Prevents runaway model, vector DB, and GPU spend |

## Common Trap

Do not treat AI apps as only prompt engineering. Production AI systems are backend systems with model-specific failure modes.

## Project Connection

Apply this folder to [Production AI Platform](../12-Projects/production-ai-platform.md), [SaaS-Level AI Product](../12-Projects/saas-level-ai-product.md), and [Enterprise RAG Platform](../12-Projects/enterprise-rag-platform.md).
