# Kubernetes, Helm, And GitOps

## 1. Problem Statement

When applications grow beyond one server, teams need orchestration:

- run multiple replicas
- restart failed containers
- route traffic
- scale services
- roll out new versions
- isolate environments
- manage config and secrets

Kubernetes solves container orchestration. Helm packages Kubernetes applications. GitOps makes Git the source of truth for deployment.

## 2. Beginner Explanation

| Topic | Notes |
| --- | --- |
| Kubernetes | Orchestrates containers across a cluster. |
| Pod | Smallest deployable unit in Kubernetes. |
| Deployment | Manages desired replicas and rollout. |
| Service | Stable internal network endpoint. |
| Ingress | Routes external traffic into services. |
| ConfigMap | Non-secret configuration. |
| Secret | Sensitive configuration object. |
| Helm | Package manager and templating system for Kubernetes. |
| GitOps | Deployment model where Git stores desired state and a controller applies it. |
| Interview Answer | Kubernetes runs and manages containers; Helm packages Kubernetes resources; GitOps uses Git and a controller like ArgoCD to keep clusters in the desired state. |

## 3. Kubernetes Core Concepts

```text
Cluster
  |
  +-- Namespace
  |     |
  |     +-- Deployment
  |     |     |
  |     |     +-- ReplicaSet
  |     |           |
  |     |           +-- Pods
  |     |
  |     +-- Service
  |     +-- Ingress
  |     +-- ConfigMap
  |     +-- Secret
```

Problems solved:

```text
Scaling
Self healing
Rolling updates
Load balancing
High availability
Service discovery
```

## 4. Kubernetes Security

Cluster security controls:

```text
Network Policies
Pod Security Standards
Admission Controllers
Runtime Detection
RBAC
Secrets integration
Certificates
```

Tools:

| Tool | Purpose |
| --- | --- |
| Kyverno | Policy-as-code and admission control |
| Falco | Runtime threat detection |
| cert-manager | TLS certificate automation |
| External Secrets Operator | Sync external secrets into Kubernetes |

## 5. Kubernetes Hardening

Recommended baseline:

- use namespaces for isolation
- enforce RBAC least privilege
- run containers as non-root
- set resource requests and limits
- use readiness and liveness probes
- enforce network policies
- restrict privileged containers
- verify signed images
- keep secrets outside Git
- enable audit logs
- monitor pod restarts and policy violations

## 6. Helm

Helm packages Kubernetes manifests.

```text
Chart
  |
  +-- templates/
  +-- values.yaml
  +-- Chart.yaml
```

Use Helm when:

- deploying the same app to dev/staging/prod
- sharing reusable Kubernetes packages
- managing releases and rollbacks

Avoid Helm when:

- a few plain manifests are enough
- the team does not understand templating
- templates become too clever

## 7. GitOps

Traditional deployment:

```text
GitHub Actions
        |
        v
kubectl apply
        |
        v
Cluster
```

GitOps deployment:

```text
GitHub
   |
   v
GitOps Repo
   |
   v
ArgoCD / FluxCD
   |
   v
Cluster
```

Benefits:

- auditable
- reproducible
- secure
- drift detection
- easier rollback
- clear deployment history

## 8. ArgoCD

Core concepts:

```text
Application
Sync
Self-healing
Promotion
Drift detection
```

ArgoCD watches Git and synchronizes the cluster to match desired state.

```text
GitOps repo changes
        |
        v
ArgoCD detects change
        |
        v
ArgoCD applies manifests
        |
        v
Cluster reaches desired state
```

## 9. FluxCD

FluxCD is another GitOps tool. It is Kubernetes-native and lightweight.

Use ArgoCD if you want:

- visual UI
- easy portfolio demos
- strong application model

Use FluxCD if you want:

- lightweight GitOps controllers
- strong Kubernetes-native automation

## 10. Environment Layout

```text
gitops-repo/
  apps/
    fastapi/
      base/
      overlays/
        dev/
        staging/
        prod/
```

Promotion flow:

```text
Update dev image digest
  |
  v
Validate dev
  |
  v
Copy same digest to staging
  |
  v
Validate staging
  |
  v
Copy same digest to prod
```

## 11. Progressive Delivery

Progressive delivery reduces deployment risk.

Options:

- rolling deployments
- canary releases
- blue-green deployments
- automated rollback

Canary example:

```text
New version gets 5 percent traffic
  |
  v
Check metrics
  |
  v
Increase to 25 percent
  |
  v
Check metrics
  |
  v
Increase to 100 percent
```

## 12. FastAPI Kubernetes Architecture

```text
Internet
  |
  v
Ingress / Load Balancer
  |
  v
FastAPI Service
  |
  v
FastAPI Pods
  |
  +-- PostgreSQL
  +-- Redis
  +-- External APIs
  +-- Observability
```

## 13. Kubernetes Manifest Example

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: fastapi-api
spec:
  replicas: 3
  selector:
    matchLabels:
      app: fastapi-api
  template:
    metadata:
      labels:
        app: fastapi-api
    spec:
      containers:
        - name: api
          image: example.com/fastapi-api:1.0.0
          ports:
            - containerPort: 8000
          readinessProbe:
            httpGet:
              path: /ready
              port: 8000
          livenessProbe:
            httpGet:
              path: /health
              port: 8000
          resources:
            requests:
              cpu: "250m"
              memory: "256Mi"
            limits:
              cpu: "500m"
              memory: "512Mi"
```

## 14. Common Mistakes

| Mistake | Fix |
| --- | --- |
| No readiness probe | Add `/ready` and only receive traffic when ready |
| Running as root | Set security context and non-root user |
| No resource limits | Define CPU and memory requests/limits |
| Direct `kubectl apply` from CI for everything | Use GitOps for production environments |
| Storing secrets in Git | Use external secret managers |
| No network policies | Restrict pod-to-pod communication |
| Scaling API without DB planning | Use pooling and monitor DB saturation |

## 15. Interview Preparation

| Question | Model Answer |
| --- | --- |
| What problem does Kubernetes solve? | It orchestrates containers with scheduling, scaling, service discovery, health checks, self-healing, and rollouts. |
| What is Helm? | Helm packages Kubernetes resources into reusable charts with templates and values. |
| What is GitOps? | GitOps stores desired deployment state in Git, and a controller like ArgoCD syncs the cluster to match it. |
| Why use ArgoCD? | It provides auditable, reproducible deployments, drift detection, self-healing, and clear environment promotion. |
| How harden Kubernetes for FastAPI? | Use RBAC, non-root containers, resource limits, probes, network policies, signed images, external secrets, and runtime monitoring. |

Are you ready for the next section?

