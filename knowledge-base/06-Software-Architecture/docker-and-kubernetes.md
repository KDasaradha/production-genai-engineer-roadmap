# Docker and Kubernetes

## 1. Problem Statement

Docker and Kubernetes solve the problem of packaging, deploying, scaling, and operating backend services consistently.

AI applications are not only notebooks or scripts. A production GenAI system may include a FastAPI API, background workers, Redis, PostgreSQL, vector databases, model gateways, local model servers, and monitoring. These services must run reliably across local, staging, and production environments.

Without Docker and orchestration:

- "works on my machine" bugs happen often
- deployments are inconsistent
- scaling workers and APIs is harder
- health checks and rollbacks are manual
- production debugging is messy

Real-world analogy: Docker is a shipping container for your app. Kubernetes is the port system that schedules, routes, restarts, and scales those containers.

## 2. Beginner Explanation

| Item | Notes |
| --- | --- |
| Definition | Docker packages applications into containers; Kubernetes orchestrates containers across machines. |
| Key terminology | image, container, Dockerfile, pod, deployment, service, ingress, config map, secret, probe |
| Simple explanation | Docker makes the app portable. Kubernetes runs and manages many containers. |
| Mental model | Package once, run consistently, scale when needed. |
| Easy example | Containerize a FastAPI app and run it with environment variables. |
| Use When | You need repeatable deployment, scaling, health checks, and service orchestration. |
| Avoid When | A small local prototype or managed serverless deployment is enough. |
| Advantages | Consistency, isolation, scalability, rollback, portability. |
| Tradeoffs | More operational complexity and configuration. |
| Limitations | Containers do not fix bad architecture or missing monitoring. |
| Production Example | Deploy RAG API pods, ingestion worker pods, Redis, PostgreSQL, and vector DB integrations. |
| Interview Answer | Docker packages services into reproducible containers, while Kubernetes manages deployment, scaling, networking, health checks, and rollouts. |

## 3. Intermediate Explanation

Docker concepts:

| Concept | Meaning |
| --- | --- |
| Dockerfile | instructions for building an image |
| Image | packaged app artifact |
| Container | running instance of an image |
| Volume | persistent or mounted data |
| Network | communication between containers |
| Compose | local multi-container orchestration |

Kubernetes concepts:

| Concept | Meaning |
| --- | --- |
| Pod | smallest deployable unit |
| Deployment | desired number of pod replicas |
| Service | stable internal network endpoint |
| Ingress | external routing into cluster |
| ConfigMap | non-secret configuration |
| Secret | sensitive configuration |
| Liveness probe | checks if pod should restart |
| Readiness probe | checks if pod can receive traffic |

AI deployment components:

- FastAPI API pods
- ingestion worker pods
- model gateway
- Redis
- PostgreSQL
- vector DB
- object storage
- local model serving pods if self-hosting

## 4. Advanced Explanation

Production deployment design separates responsibilities:

- API services handle user requests.
- Workers handle long-running ingestion and evaluation.
- Databases stay managed or stateful.
- Model gateways route model calls.
- Observability captures logs, metrics, and traces.

Optimization techniques:

- keep API containers stateless
- scale workers separately from APIs
- use health and readiness probes
- set CPU/memory requests and limits
- use rolling deployments
- mount config through environment variables
- use secrets management
- build small images
- pin dependency versions

Performance considerations:

- model calls may be slow, so API timeouts matter
- ingestion workers need different resources than APIs
- local model servers may need GPUs
- vector DB and PostgreSQL should not be overloaded by API scaling

Production challenges:

- wrong environment variables
- secrets leakage
- missing health checks
- worker crashes
- database connection exhaustion
- GPU scheduling
- rolling deploys breaking long streams

## 5. Internal Working

```text
Code
  |
  v
Docker image build
  |
  v
Image pushed to registry
  |
  v
Kubernetes deployment pulls image
  |
  v
Pods start and pass readiness checks
  |
  v
Service routes traffic
  |
  v
Monitoring observes health and performance
```

AI deployment lifecycle:

1. Build FastAPI image.
2. Build worker image.
3. Push images to registry.
4. Deploy API with readiness probe.
5. Deploy workers separately.
6. Connect to Redis, PostgreSQL, vector DB, and model provider.
7. Run smoke tests.
8. Monitor logs, latency, errors, and queue depth.
9. Roll back if deployment fails.

## 6. When To Use

Use Docker when:

- you want reproducible local and production environments
- your app has dependencies
- you need deployable artifacts

Use Kubernetes when:

- you need multiple replicas
- rolling deploys matter
- services need scaling
- workers and APIs scale separately
- your team can operate it

## 7. When NOT To Use

Avoid Kubernetes when:

- the app is a small prototype
- managed containers or serverless are simpler
- team lacks ops experience
- operational overhead exceeds product value

Better alternatives:

- Docker Compose for local development
- managed container platforms
- serverless functions for simple APIs
- PaaS deployments

## 8. Advantages

- Consistent deployments.
- Easier scaling.
- Health checks and restarts.
- Rolling updates.
- Environment isolation.
- Strong portfolio signal for production readiness.

## 9. Tradeoffs

| Tradeoff | Explanation |
| --- | --- |
| Control vs complexity | Kubernetes gives control but adds operational burden. |
| Portability vs configuration | Containers are portable, but config still matters. |
| Scaling API vs dependencies | Scaling pods can overload DB/model providers. |
| Self-hosting vs managed services | Self-hosting gives control but increases responsibility. |

## 10. Limitations

- Kubernetes does not solve application bugs.
- Stateful services need careful operations.
- GPU workloads require special scheduling.
- Bad resource limits can cause instability.
- Secrets must still be managed securely.

## 11. Real-World Examples

Startup example: Docker Compose runs FastAPI, Redis, PostgreSQL, and a vector DB locally.

Enterprise example: Kubernetes deploys API pods and worker pods with separate autoscaling rules.

FAANG-style example: internal platforms deploy AI services with service mesh, canaries, telemetry, and automated rollback.

Production system: a RAG platform deploys stateless API pods, ingestion workers, managed PostgreSQL, Redis, vector DB, and a model gateway.

## 12. Architecture Diagram

```text
[Internet]
    |
    v
[Ingress / Load Balancer]
    |
    v
[FastAPI API Pods] ----> [Redis]
    |                    [PostgreSQL]
    |                    [Vector DB]
    |
    v
[Worker Pods] --------> [Object Storage]
    |
    v
[Model Gateway / LLM Provider]
```

## 13. Python Implementation

Health endpoint:

```python
from fastapi import FastAPI

app = FastAPI()

@app.get("/health")
async def health() -> dict[str, str]:
    return {"status": "ok"}

@app.get("/ready")
async def ready() -> dict[str, str]:
    # In production, check database, Redis, and critical dependencies.
    return {"status": "ready"}
```

Settings model:

```python
from pydantic import BaseModel

class Settings(BaseModel):
    database_url: str
    redis_url: str
    environment: str
```

## 14. FastAPI Implementation

Dockerfile example:

```dockerfile
FROM python:3.12-slim

WORKDIR /app

COPY pyproject.toml .
RUN pip install --no-cache-dir fastapi uvicorn

COPY app ./app

CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000"]
```

Kubernetes deployment idea:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: ai-api
spec:
  replicas: 3
  selector:
    matchLabels:
      app: ai-api
  template:
    metadata:
      labels:
        app: ai-api
    spec:
      containers:
        - name: api
          image: ai-api:latest
          ports:
            - containerPort: 8000
          readinessProbe:
            httpGet:
              path: /ready
              port: 8000
```

Production-ready structure:

```text
deployment/
  Dockerfile
  docker-compose.yml
  k8s/
    api-deployment.yaml
    api-service.yaml
    worker-deployment.yaml
    ingress.yaml
```

## 15. Database Integration

PostgreSQL:

- use managed DB when possible
- configure connection pooling
- run migrations before or during deployment carefully
- readiness checks should verify DB access

Redis:

- configure connection limits
- use managed Redis for production when possible
- do not store critical permanent data only in Redis

Vector DB:

- avoid rebuilding indexes during API startup
- run ingestion through workers

## 16. Production Considerations

- Add `/health` and `/ready`.
- Keep containers stateless.
- Set resource requests and limits.
- Use secrets manager, not hardcoded secrets.
- Use rolling updates.
- Separate API and worker deployments.
- Monitor pod restarts.
- Configure graceful shutdown for streaming.
- Use connection pools.
- Run smoke tests after deployment.

## 17. Common Mistakes

| Level | Mistake | How To Avoid |
| --- | --- | --- |
| Beginner | No Dockerfile | Containerize the app |
| Beginner | Hardcoding config | Use environment variables and secrets |
| Intermediate | No readiness probe | Add `/ready` checks |
| Intermediate | API and worker in same process | Deploy and scale separately |
| Production | No resource limits | Set CPU/memory requests and limits |
| Production | Scaling API until DB fails | monitor dependency saturation |

## 18. Interview Preparation

| Level | Question | Model Answer |
| --- | --- | --- |
| Basic | What is Docker? | A tool for packaging applications and dependencies into containers. |
| Basic | What is Kubernetes? | A system for running, scaling, networking, and managing containers. |
| Intermediate | Why separate API and worker deployments? | They have different workloads, scaling needs, and failure patterns. |
| Advanced | How deploy a production RAG system? | Containerize API and workers, use managed DB/cache/vector services, add probes, secrets, scaling, logging, and deployment rollback. |
| Scenario | Pods restart repeatedly. | Check logs, probes, memory limits, environment variables, dependency connectivity, and startup failures. |

## 19. System Design Discussion

Docker and Kubernetes are deployment tools. They should reflect application boundaries:

- API
- worker
- model gateway
- model server
- database
- cache
- vector store
- observability

Design decisions:

- Docker Compose vs Kubernetes
- managed DB vs in-cluster DB
- API autoscaling metric
- worker autoscaling metric
- secrets strategy
- rollout strategy
- GPU scheduling for local models

## 20. Hands-On Assignment

- Easy: Write a Dockerfile for a FastAPI app.
- Medium: Add `/health` and `/ready` endpoints.
- Hard: Design separate Kubernetes deployments for API and ingestion workers.

## 21. Mini Project

Dockerize an AI Chat API.

Requirements:

- FastAPI app.
- Dockerfile.
- Docker Compose with Redis and PostgreSQL.
- Health and readiness endpoints.
- Environment-based config.

Folder structure:

```text
dockerized-ai-api/
  app/
    main.py
    settings.py
  deployment/
    Dockerfile
    docker-compose.yml
```

## 22. Production-Level Project

Deploy a Production RAG Platform.

Real-world problem:

- A company needs a scalable RAG API with document ingestion and monitored deployment.

Architecture:

```text
[Ingress] -> [FastAPI API Pods]
          -> [Worker Pods]
          -> [Managed PostgreSQL]
          -> [Managed Redis]
          -> [Vector DB]
          -> [LLM Provider]
```

Tech stack:

- Docker
- Kubernetes or managed containers
- FastAPI
- PostgreSQL
- Redis
- vector database
- object storage
- monitoring stack

Scaling strategy:

- autoscale API by CPU/request rate
- autoscale workers by queue depth
- use managed stateful services
- add rolling deploys
- add smoke tests and rollback

## Quiz

1. What problem does Docker solve?
2. What problem does Kubernetes solve?
3. What is a Docker image?
4. What is a Kubernetes pod?
5. What is a readiness probe?
6. Why separate API and worker deployments?
7. Why should containers be stateless?
8. What are resource limits?
9. How can API scaling overload a database?
10. How would you deploy a RAG app with workers?

## Knowledge Check

You should be able to explain Docker and Kubernetes deployment for production AI systems, including APIs, workers, health checks, secrets, scaling, and rollback.

Are you ready for the next section?
