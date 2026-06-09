# Cloud, Registries, And Tooling

## 1. Problem Statement

Production systems need a place to run, a place to store artifacts, and tools to package and configure deployment.

The core choices are:

```text
Cloud platform
Container registry
Kubernetes platform
GitOps tool
Package manager
Deployment templating tool
Database
Security maturity level
```

Choosing poorly can increase cost, complexity, lock-in, and hiring difficulty.

## 2. Beginner Explanation

| Topic | Notes |
| --- | --- |
| Cloud platform | Provides compute, networking, storage, databases, identity, and managed services. |
| Container registry | Stores Docker images before deployment. |
| Kubernetes platform | Runs containerized workloads across nodes. |
| GitOps tool | Syncs desired deployment state from Git into the cluster. |
| Package manager | Manages application dependencies. |
| Deployment tooling | Packages or customizes Kubernetes manifests. |
| Use When | You need production infrastructure and repeatable deployment. |
| Avoid When | A local-only project does not need cloud resources. |
| Interview Answer | Cloud and registry choices define where the app runs, where artifacts live, how deployments are promoted, and how teams secure the software supply chain. |

## 3. Cloud Options

### AWS

Best overall ecosystem and most common for backend/cloud job readiness.

Core services:

```text
EC2
EKS
ECS
RDS
ECR
IAM
S3
VPC
Load Balancer
Auto Scaling
```

Use when:

- you want the widest market relevance
- you need strong container, database, IAM, and networking options
- you want ECS before Kubernetes

Tradeoff: large surface area and many configuration details.

### Azure

Best for Microsoft-heavy organizations.

Core services:

```text
Virtual Machines
AKS
ACR
Azure SQL
Azure Container Apps
Azure Key Vault
```

Use when:

- company stack is Microsoft/Azure
- identity integrates with Microsoft Entra ID

Tradeoff: less common in some startup/backend interview tracks than AWS.

### GCP

Best Kubernetes experience and strong serverless/container developer experience.

Core services:

```text
GKE
Cloud Run
Artifact Registry
Cloud SQL
IAM
Cloud Storage
```

Use when:

- Kubernetes and Cloud Run are primary targets
- team values simple managed container workflows

Tradeoff: AWS still appears more often in many backend job descriptions.

### On-Premises

Use for:

- regulated environments
- private infrastructure
- companies with existing data centers

Tradeoff: you own more operational complexity.

## 4. Container Registry Options

| Registry | Best For | Notes |
| --- | --- | --- |
| Docker Hub | Public images and simple learning | Popular but rate limits and public exposure matter |
| GHCR | GitHub-native projects | Good with GitHub Actions and packages |
| ECR | AWS production | Best default for AWS ECS/EKS |
| ACR | Azure production | Best default for AKS and Azure workloads |
| Artifact Registry | GCP production | Best default for GKE and Cloud Run |
| Harbor | Enterprise/private registry | Strong for self-hosted registry, scanning, and governance |

Registry security basics:

- use private repositories for production images
- scan images before promotion
- sign images
- store SBOMs
- restrict push permissions
- use immutable tags or digest pinning for production

## 5. Kubernetes Options

| Option | Use When | Tradeoff |
| --- | --- | --- |
| EKS | AWS production platform | More AWS integration and configuration |
| AKS | Azure production platform | Best for Azure shops |
| GKE | Kubernetes-focused GCP platform | Strong Kubernetes UX |
| Self-managed Kubernetes | Full control or on-prem needs | Highest ops burden |

Interview answer: Managed Kubernetes reduces control-plane management, but teams still own workload security, networking, resource limits, observability, upgrade planning, and cost management.

## 6. GitOps Tools

### ArgoCD

Use when:

- you want visual GitOps workflows
- teams need application sync visibility
- you want drift detection and self-healing

Core concepts:

```text
Application
Sync
Drift detection
Self-healing
Promotion
```

### FluxCD

Use when:

- you prefer a lighter GitOps operator model
- teams are comfortable with Kubernetes-native automation

Tradeoff: ArgoCD is often easier to demo and explain in interviews because of its UI and application model.

## 7. Python Package Management

| Tool | Use When | Tradeoff |
| --- | --- | --- |
| uv | You want fast modern Python dependency management | Newer ecosystem adoption |
| Poetry | You want mature project/dependency management | Can be slower or opinionated |
| pip + requirements.txt | You want simple, universal setup | Less structured dependency management |

Recommended for a modern FastAPI project: `uv` or Poetry for application projects; `pip` is still useful for simple deployments and compatibility.

## 8. Deployment Style

### Helm

Use when:

- packaging Kubernetes applications
- templating values for dev/staging/prod
- managing releases

Concepts:

```text
Chart
Template
Values
Helpers
Release
```

### Kustomize

Use when:

- you want patch-based environment customization
- you want less templating logic

### Helm + Kustomize

Use when:

- Helm packages the app
- Kustomize overlays environment-specific changes

Tradeoff: combining tools can increase cognitive load.

## 9. Database Options

| Database | Use When | Tradeoff |
| --- | --- | --- |
| PostgreSQL | Default production relational database | Requires tuning, migrations, backups |
| MySQL | Existing ecosystem or team preference | Different feature set and tuning model |
| Aurora PostgreSQL | AWS-managed scale and availability | More AWS-specific and cost-sensitive |

Recommended for your FastAPI path: PostgreSQL first, RDS/Aurora PostgreSQL for AWS production.

## 10. Security And Compliance Maturity Levels

| Level | Characteristics |
| --- | --- |
| Startup Production | Docker, CI/CD, basic scans, managed DB, basic monitoring |
| Growth Stage | Kubernetes or ECS, GitOps, image scanning, secrets manager, alerts |
| SOC2 Ready | audit logs, access controls, change management, incident response, evidence collection |
| ISO27001 Ready | formal information security management, risk treatment, documentation, internal controls |
| FinTech / Banking Grade | strong identity, least privilege, policy-as-code, signed artifacts, runtime security, DR, auditability |

## 11. Recommended Stack For You

```text
FastAPI
PostgreSQL
Docker
AWS
ECR
ECS first
EKS later
Helm
GitHub Actions
ArgoCD
```

Security:

```text
Semgrep
Bandit
Gitleaks
Trivy
Syft
Cosign
Kyverno
Falco
```

Observability:

```text
OpenTelemetry
Prometheus
Grafana
Loki
```

Secrets:

```text
AWS Secrets Manager
External Secrets Operator
```

## 12. Common Mistakes

| Mistake | Fix |
| --- | --- |
| Choosing tools by hype | Choose by workload, team skill, company stage, and hiring goals |
| Using Docker Hub for private production by default | Prefer cloud-native private registries such as ECR/ACR/Artifact Registry |
| Running Kubernetes before knowing cloud basics | Learn IAM, networking, compute, storage, and registry first |
| Ignoring package reproducibility | Pin dependencies and use lock files where possible |
| Treating compliance as a checklist at the end | Design auditability, access control, and evidence collection early |

## 13. Interview Preparation

| Question | Model Answer |
| --- | --- |
| Why choose AWS for learning? | AWS has broad industry adoption and teaches identity, networking, compute, storage, databases, registries, and managed containers in a job-relevant way. |
| What is a container registry? | A registry stores container images so CI/CD and runtimes can pull versioned application artifacts. |
| GHCR vs ECR? | GHCR integrates well with GitHub. ECR is the best default for AWS ECS/EKS production because IAM and networking integrate natively. |
| Helm vs Kustomize? | Helm templates and packages apps as charts. Kustomize patches plain manifests for environment-specific configuration. |
| ArgoCD vs FluxCD? | Both implement GitOps. ArgoCD offers a strong application model and UI; FluxCD is lightweight and Kubernetes-native. |

Are you ready for the next section?

