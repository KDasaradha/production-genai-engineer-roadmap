# DevOps and DevSecOps

This folder turns `devops-notes.md` into organized learning notes for becoming capable of taking applications from source code to secure production operation.

The source file is intentionally left unchanged. Duplicate roadmap and pipeline explanations were consolidated here into one clear curriculum.

## Learning Goal

By the end of this section, you should be able to:

- Containerize Python, FastAPI, React, Next.js, and Java applications.
- Build CI/CD pipelines with GitHub Actions.
- Deploy to AWS EC2, ECS, and EKS.
- Operate Kubernetes workloads with Helm and ArgoCD.
- Implement DevSecOps with SAST, SCA, secrets scanning, SBOMs, image signing, policy enforcement, and runtime security.
- Add observability with metrics, logs, traces, dashboards, and alerts.
- Design production-grade deployment architectures with rollback, scaling, incident response, backup, and compliance readiness.

## Learning Order

| Order | File | Outcome |
| --- | --- | --- |
| 1 | [01-platform-engineer-roadmap.md](01-platform-engineer-roadmap.md) | Understand the job-oriented roadmap and final capability target |
| 2 | [02-deployment-fundamentals.md](02-deployment-fundamentals.md) | Learn deployment options from direct servers to Kubernetes and PaaS |
| 3 | [03-cloud-registries-and-tooling.md](03-cloud-registries-and-tooling.md) | Compare AWS, Azure, GCP, registries, databases, package tools, and deployment tooling |
| 4 | [04-cicd-github-actions.md](04-cicd-github-actions.md) | Design PR validation, build, promotion, release, and environment workflows |
| 5 | [05-devsecops-supply-chain-security.md](05-devsecops-supply-chain-security.md) | Secure code, dependencies, containers, artifacts, and delivery pipelines |
| 6 | [06-kubernetes-helm-gitops.md](06-kubernetes-helm-gitops.md) | Learn Kubernetes, Helm, GitOps, ArgoCD, FluxCD, and cluster hardening |
| 7 | [07-production-platform-engineering.md](07-production-platform-engineering.md) | Build production standards for secrets, threat modeling, DB security, cost, performance, and docs |
| 8 | [08-observability-sre-compliance.md](08-observability-sre-compliance.md) | Add observability, incident response, backup, disaster recovery, SRE, SOC2, and ISO readiness |
| 9 | [09-deployment-strategies.md](09-deployment-strategies.md) | Compare recreate, rolling, blue-green, canary, A/B, shadow, and feature flag deployments |
| 10 | [10-portfolio-projects-and-checklists.md](10-portfolio-projects-and-checklists.md) | Convert the learning path into portfolio projects and enterprise checklists |

## Core Mental Model

```text
Developer gives code
        |
        v
You build CI/CD
        |
        v
You containerize it
        |
        v
You deploy it
        |
        v
You secure it
        |
        v
You monitor it
        |
        v
You scale it
        |
        v
You operate it in production
```

## Recommended Stack For A Production FastAPI Platform

| Layer | Recommended Choices |
| --- | --- |
| Backend | FastAPI, PostgreSQL, Redis |
| Packaging | Docker, Helm |
| Source control | GitHub, branch protection, CODEOWNERS |
| CI/CD | GitHub Actions, reusable workflows, protected environments |
| Registry | ECR, GHCR, Harbor, or ACR |
| Cloud | AWS first for employability, then Azure/GCP comparison |
| Runtime | ECS Fargate first, EKS when Kubernetes is needed |
| GitOps | ArgoCD, optionally FluxCD |
| Security | Ruff, Mypy, Bandit, Semgrep, Gitleaks, Trivy, Syft, Cosign |
| Platform security | Kyverno, External Secrets Operator, cert-manager, Falco |
| Observability | OpenTelemetry, Prometheus, Grafana, Loki, Alertmanager |
| Compliance | SOC2, ISO27001, threat modeling, incident response, audit logs |

## Interview Answer

DevOps is the discipline of delivering software reliably through automation, infrastructure, deployment, monitoring, and operations. DevSecOps extends that by embedding security into every stage of the software lifecycle: development, pull requests, build, artifact registry, deployment, runtime, compliance, and incident response.

Are you ready for the next section?

