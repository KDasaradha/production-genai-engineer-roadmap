# Deployment Fundamentals

## 1. Problem Statement

Deployment is the process of moving an application from source code into an environment where users can access it.

Without a proper deployment model:

- code runs differently across machines
- releases are manual and inconsistent
- rollback is slow
- scaling is unclear
- secrets and config are mishandled
- debugging production is painful

Real-world analogy: development is cooking a recipe; deployment is running a restaurant kitchen where the same meal must be served reliably every day.

## 2. Beginner Explanation

| Topic | Notes |
| --- | --- |
| Definition | Deployment means releasing application code into a runtime environment. |
| Key terms | VM, container, registry, environment, load balancer, rollback, scaling |
| Mental model | Code must become a running service with config, networking, storage, health checks, and monitoring. |
| Use When | Any application must be used outside your laptop. |
| Avoid When | You are only testing local code temporarily. |
| Advantages | Repeatability, reliability, user access, scaling, rollback. |
| Tradeoffs | Each deployment option has different complexity, cost, and control. |
| Limitations | Deployment does not replace testing, security, monitoring, or architecture. |
| Interview Answer | Deployment is the controlled process of releasing tested application artifacts into an environment with configuration, networking, health checks, monitoring, and rollback. |

## 3. Deployment Levels

| Level | Option | Best For | Main Tradeoff |
| --- | --- | --- | --- |
| 1 | Direct server, no Docker | Learning Linux and simple apps | Manual setup and environment drift |
| 2 | Docker on one server | Small production apps | Single-server limitation |
| 3 | Docker Compose | Multi-service local or small deployments | Not ideal for large-scale production |
| 4 | Managed containers | Production containers without Kubernetes | Less low-level control |
| 5 | Kubernetes | Cloud-native scaling and orchestration | Highest operational complexity |
| 6 | Serverless containers | Event-driven or bursty workloads | Platform constraints and cold starts |
| 7 | PaaS | Fast startup/product deployments | Less control and possible vendor lock-in |

## 4. Level 1: Direct VM Deployment

```text
Developer
   |
   v
SSH into server
   |
   v
Install Python/Node/Java
   |
   v
Run app with systemd
   |
   v
Nginx exposes app
```

Pros:

- simple mental model
- teaches Linux, processes, logs, ports, and Nginx
- useful for early learning

Cons:

- hard to reproduce
- upgrades can break the server
- scaling requires manual work
- rollback is weak

Use for: first FastAPI Linux deployment.

Avoid for: serious multi-service production.

## 5. Level 2: Docker On A Single Server

```text
Code -> Docker image -> Server -> Docker container
```

Pros:

- repeatable application packaging
- simpler dependency management
- easier rollback by running a previous image

Cons:

- single server is still a bottleneck
- service discovery and scaling remain manual
- monitoring and backups still need separate setup

Use for: small apps and portfolio demos.

Avoid for: high availability requirements.

## 6. Level 3: Docker Compose

```text
docker-compose.yml
   |
   +-- FastAPI
   +-- PostgreSQL
   +-- Redis
   +-- Next.js
```

Pros:

- excellent for local multi-service development
- easy to run app + database + cache
- good stepping stone before Kubernetes

Cons:

- limited production orchestration
- weaker self-healing and scaling
- deployments are usually server-centric

Use for: local development, internal tools, small VPS setups.

Avoid for: enterprise-grade scaling and HA.

## 7. Level 4: Managed Container Services

### AWS ECS Fargate

Core concepts:

```text
Task Definition
Service
Load Balancer
Autoscaling
```

Pros:

- production containers without Kubernetes complexity
- strong AWS integration
- no server management
- good for FastAPI, React, Next.js, Spring Boot

Cons:

- AWS-specific
- less portable than Kubernetes
- advanced networking can still be tricky

### Azure Container Apps

Best for teams already using Azure and Microsoft ecosystems.

### Google Cloud Run

Best for simple containerized services with automatic scaling.

Pros:

- simple deployment
- scales down when idle
- strong developer experience

Cons:

- platform constraints
- cold start considerations
- not ideal for all long-running workloads

## 8. Level 5: Kubernetes

Managed Kubernetes options:

| Cloud | Service |
| --- | --- |
| AWS | EKS |
| Azure | AKS |
| GCP | GKE |
| Self-managed | Kubernetes on your own servers |

Problems Kubernetes solves:

```text
Scaling
Self-healing
Rolling updates
Load balancing
High availability
Service discovery
Config and secret distribution
```

Pros:

- powerful orchestration
- strong ecosystem
- portable deployment model
- supports GitOps and platform engineering

Cons:

- operational complexity
- security hardening required
- debugging requires experience
- can be expensive if poorly managed

## 9. Level 6: Serverless Containers

Example:

```text
AWS Lambda container image
```

Pros:

- low ops overhead
- good for event-driven workloads
- scales automatically

Cons:

- runtime limits
- cold starts
- not ideal for every API or long-running process

## 10. Level 7: PaaS

Examples:

```text
Render
Railway
Fly.io
```

Pros:

- fastest deployment experience
- great for early projects
- minimal infrastructure setup

Cons:

- less control
- scaling and networking constraints
- vendor-specific behavior

## 11. Local Deployment Options

| Option | Use |
| --- | --- |
| Docker Desktop | Run containers locally |
| Kubernetes in Docker Desktop | Lightweight local Kubernetes learning |
| Minikube | Local Kubernetes cluster for learning |
| Kind | Kubernetes in Docker, useful for testing clusters |

## 12. Complete Deployment Landscape

```text
No Docker
  |
Docker single server
  |
Docker Compose
  |
Managed containers
  |
Kubernetes
  |
GitOps + platform controls
```

## 13. Recommendation By Company Size

| Company Stage | Recommended Deployment |
| --- | --- |
| Personal learning | Docker Compose, then one cloud VM |
| Startup MVP | PaaS, Cloud Run, or ECS Fargate |
| Growing startup | ECS/EKS with managed databases |
| Enterprise | Kubernetes, GitOps, policy, observability, compliance |
| FinTech/Banking | Kubernetes with strict supply chain, runtime security, audit, DR |

## 14. FastAPI Platform Recommendation

For a serious FastAPI platform:

```text
FastAPI
PostgreSQL
Redis
Docker
GitHub Actions
ECR
ECS first
EKS later
ArgoCD when Kubernetes is introduced
```

## 15. Common Mistakes

| Mistake | Fix |
| --- | --- |
| Jumping straight to Kubernetes | Learn direct VM, Docker, Compose, and managed containers first |
| Treating deployment as only `docker run` | Include config, secrets, health checks, monitoring, and rollback |
| Running databases casually in containers for production | Prefer managed databases for real production |
| No rollback strategy | Keep versioned images and release history |
| No health checks | Add `/health` and `/ready` endpoints |

## 16. Interview Preparation

| Question | Model Answer |
| --- | --- |
| What is deployment? | Deployment is releasing a tested artifact into a runtime environment with configuration, networking, health checks, observability, and rollback. |
| Docker Compose vs Kubernetes? | Compose is simple multi-container orchestration, mostly local or small-scale. Kubernetes is a production orchestration platform for scaling, self-healing, networking, and rollouts. |
| When would you choose ECS over Kubernetes? | When the team uses AWS and wants production containers without Kubernetes operational complexity. |
| When is PaaS better? | For early products where speed matters more than infrastructure control. |
| Why avoid direct VM deployment for serious production? | It is harder to reproduce, scale, audit, and roll back. |

Are you ready for the next section?

