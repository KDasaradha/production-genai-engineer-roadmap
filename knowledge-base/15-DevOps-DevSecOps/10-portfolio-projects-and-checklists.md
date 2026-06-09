# Portfolio Projects And Checklists

## 1. Problem Statement

DevOps and DevSecOps skills become job-ready only when you can demonstrate them through working projects and explain the production decisions.

The goal is not to memorize tools. The goal is:

```text
I can take any application from source code to production securely.
```

## 2. Portfolio Project Ladder

| Project | Stack | Skills Demonstrated |
| --- | --- | --- |
| Project 1 | FastAPI + PostgreSQL + Docker + GitHub Actions + EC2 | Linux, Docker, CI, cloud basics |
| Project 2 | Next.js + FastAPI + Docker Compose + GitHub Actions | multi-service app delivery |
| Project 3 | Spring Boot + PostgreSQL + Docker + ECS | Java deployment and managed containers |
| Project 4 | FastAPI + Spring Boot + React + ECR + EKS + Helm + ArgoCD | Kubernetes, GitOps, microservices |
| Project 5 Capstone | FastAPI + Next.js + PostgreSQL + EKS + DevSecOps + Observability + Platform security | enterprise DevSecOps platform |

## 3. Capstone Requirements

Build one platform:

```text
Frontend
  Next.js

Backend
  FastAPI

Database
  PostgreSQL

Container
  Docker

CI/CD
  GitHub Actions

Registry
  ECR

Runtime
  EKS

Deployment
  Helm

GitOps
  ArgoCD

Security
  Semgrep
  Gitleaks
  Trivy
  Syft
  Cosign

Observability
  OpenTelemetry
  Prometheus
  Grafana
  Loki

Platform
  Secrets Manager
  External Secrets Operator
  Kyverno
  Falco
  cert-manager
```

## 4. Enterprise Pipeline Checklist

### Infrastructure

- AWS account/project structure
- IAM roles and least privilege
- VPC/subnets/security groups
- ECS or EKS runtime
- RDS PostgreSQL
- Redis if needed
- S3/object storage if needed

### Container Registry

- ECR/GHCR/ACR/Harbor
- private repositories
- vulnerability scanning
- signed images
- immutable tags or digest-based deployment

### Kubernetes

- namespaces
- deployments
- services
- ingress
- config maps
- external secrets
- resource requests/limits
- probes
- network policies
- RBAC

### GitOps

- separate GitOps repo
- dev/staging/prod overlays
- ArgoCD applications
- drift detection
- promotion by image digest

### Packaging

- Dockerfile
- Helm chart
- values per environment
- versioned images

### Deployment Style

- rolling for normal releases
- canary for risky releases
- blue-green for major changes
- rollback plan

### Database

- managed PostgreSQL
- migrations
- backups
- restore testing
- connection pooling
- least privilege users

### Security Level

- pre-commit hooks
- SAST
- SCA
- secrets scanning
- container scanning
- IaC scanning
- SBOM
- artifact signing
- policy-as-code
- runtime security
- audit logs

## 5. Recommended Scale-Up Path

```text
1. Local FastAPI + PostgreSQL
2. Dockerize FastAPI
3. Add Docker Compose
4. Add GitHub Actions tests/lint
5. Push image to registry
6. Deploy to EC2 or ECS
7. Add SAST/SCA/secrets scan
8. Add SBOM and signing
9. Move to Kubernetes
10. Package with Helm
11. Deploy with ArgoCD
12. Add observability
13. Add Kyverno/Falco/External Secrets
14. Add incident response and compliance evidence
```

## 6. What I Would Implement For A Serious FastAPI Project

```text
FastAPI
PostgreSQL
Docker
GitHub
GitHub Actions
Ruff + Mypy + Bandit + Gitleaks
Semgrep
Trivy
Syft
Cosign
GHCR or Harbor or ECR
Kubernetes
ArgoCD
Kyverno
External Secrets Operator
cert-manager
OpenTelemetry
Prometheus + Grafana + Loki
Falco
```

This creates a supply-chain-secure DevSecOps platform, not just a basic CI/CD setup.

## 7. Portfolio Proof Checklist

For every project, write:

- architecture diagram
- deployment diagram
- CI/CD workflow explanation
- security gates
- rollback strategy
- monitoring dashboard screenshots or descriptions
- incident runbook
- cost considerations
- interview explanation

## 8. Interview Story Template

Use this structure:

```text
Problem:
  The app needed reliable secure deployment.

Architecture:
  GitHub -> GitHub Actions -> Docker -> Registry -> ECS/EKS -> Observability.

Security:
  SAST, SCA, secrets scan, image scan, SBOM, signing, policy checks.

Operations:
  Health checks, logs, metrics, traces, alerts, rollback.

Tradeoffs:
  Chose ECS for lower ops or EKS for portability/GitOps/platform controls.

Result:
  Repeatable deployment with safer releases and clear debugging.
```

## 9. Common Mistakes

| Mistake | Fix |
| --- | --- |
| Building only toy deployments | Add CI/CD, registry, secrets, monitoring, and rollback |
| No written explanation | Create README diagrams and interview answers |
| Skipping security | Add scans, SBOM, signing, and secret management |
| No production tradeoffs | Explain why ECS vs EKS, Helm vs Kustomize, rolling vs canary |
| Project too large to finish | Build in phases and document each milestone |

## 10. Final Knowledge Check

You are ready for DevOps/DevSecOps interviews when you can explain:

- Linux process and network basics
- Git branch and release strategy
- Docker images, containers, volumes, networks, registries
- Docker Compose for multi-service apps
- GitHub Actions workflows, jobs, steps, artifacts, cache, secrets, environments
- AWS IAM, VPC, EC2, RDS, S3, ECR, ECS, EKS
- Kubernetes pods, deployments, services, ingress, config maps, secrets, HPA
- Helm charts and environment values
- ArgoCD GitOps promotion
- SAST, SCA, secrets scanning, IaC scanning, container scanning
- SBOMs, Cosign signing, SLSA, policy-as-code
- OpenTelemetry, Prometheus, Grafana, Loki
- incident response, RPO, RTO, SLI, SLO, SLA, error budgets
- SOC2 and ISO27001 readiness basics

Are you ready for the next section?

