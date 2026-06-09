# Deployment Strategies

## 1. Problem Statement

A deployment strategy controls how a new version reaches users.

The goal is to reduce risk while releasing changes.

Without a clear strategy:

- users may see broken releases
- rollback is slow
- downtime may occur
- teams cannot test production safely
- failures impact everyone at once

## 2. Beginner Explanation

| Topic | Notes |
| --- | --- |
| Recreate | Stop old version, start new version. |
| Rolling | Replace instances gradually. |
| Blue-green | Run old and new environments, then switch traffic. |
| Canary | Send a small percentage of traffic to new version first. |
| A/B | Route users to variants for product experimentation. |
| Shadow | Send copied traffic to new version without user impact. |
| Feature flag | Turn features on/off without redeploying. |
| Interview Answer | Deployment strategies control rollout risk by deciding how traffic moves from old code to new code. |

## 3. Recreate Deployment

```text
Stop old version
        |
        v
Start new version
```

Use when:

- downtime is acceptable
- app is internal or low-risk
- infrastructure is simple

Avoid when:

- user-facing downtime is unacceptable
- rollback must be fast

Advantages:

- simplest strategy
- no two-version compatibility issues

Tradeoffs:

- downtime
- high release risk

## 4. Rolling Deployment

```text
Old pods: A A A A
        |
        v
Replace one by one
        |
        v
New pods: B B B B
```

Use when:

- normal web/API workloads
- zero or near-zero downtime is needed
- changes are backward compatible

Advantages:

- common default in Kubernetes
- no full duplicate environment required
- lower downtime

Tradeoffs:

- old and new versions run together temporarily
- database changes must be compatible

## 5. Blue-Green Deployment

```text
Blue: current production
Green: new version
        |
        v
Switch traffic from Blue to Green
```

Use when:

- fast rollback is important
- production-like validation is needed before traffic switch

Advantages:

- quick rollback
- clean separation between versions

Tradeoffs:

- requires duplicate infrastructure
- database migration compatibility still matters

## 6. Canary Deployment

```text
New version gets 5 percent traffic
        |
        v
Monitor
        |
        v
Increase to 25 percent
        |
        v
Increase to 50 percent
        |
        v
Increase to 100 percent
```

Use when:

- release risk is high
- you have good metrics and rollback automation
- user impact must be limited

Advantages:

- limits blast radius
- validates with real traffic
- supports automated rollback

Tradeoffs:

- needs traffic routing and observability
- can be complex with stateful changes

## 7. A/B Deployment

```text
Users group A -> Version A
Users group B -> Version B
```

Use when:

- testing product behavior
- measuring conversion or engagement

Avoid when:

- goal is only safe infrastructure rollout

Tradeoff: A/B testing is product experimentation, not primarily reliability deployment.

## 8. Shadow Deployment

```text
Real traffic -> Current version -> User response
           |
           +-> Shadow version -> No user response
```

Use when:

- testing a new service with production-like traffic
- validating performance without user impact

Tradeoffs:

- duplicate processing cost
- must avoid side effects such as duplicate writes/emails/payments

## 9. Feature Flag Deployment

```text
Deploy code hidden
        |
        v
Enable feature for internal users
        |
        v
Enable for beta users
        |
        v
Enable for everyone
```

Use when:

- separating deployment from release
- enabling gradual rollout
- quickly disabling risky features

Tradeoffs:

- flag cleanup is required
- too many flags create complexity

## 10. Recommended Strategy For FastAPI

| Stage | Strategy |
| --- | --- |
| Learning project | Recreate or rolling |
| Small production | Rolling |
| Important backend | Rolling with health checks and rollback |
| High-risk release | Canary |
| Major infrastructure change | Blue-green |
| Product experiment | A/B |
| Risky backend rewrite | Shadow |
| Feature launch | Feature flag |

## 11. Database Migration Rule

For rolling, canary, and blue-green deployments, database changes must be backward compatible.

Safe pattern:

```text
1. Add new nullable column/table
2. Deploy code that writes both old and new format
3. Backfill data
4. Read from new format
5. Remove old column/code later
```

## 12. Common Mistakes

| Mistake | Fix |
| --- | --- |
| Canary without metrics | Add error, latency, saturation, and business metrics |
| Blue-green with incompatible DB migration | Use backward-compatible migration stages |
| Feature flags never removed | Track owner and expiration |
| Shadow traffic causing side effects | Disable writes and external side effects |
| Rolling deploy with no readiness probe | Add readiness probes before rollout |

## 13. Interview Preparation

| Question | Model Answer |
| --- | --- |
| What is rolling deployment? | Gradually replacing old instances with new ones, reducing downtime while both versions may run temporarily. |
| Blue-green vs canary? | Blue-green switches traffic between two full environments. Canary sends a small traffic percentage to the new version and increases gradually based on metrics. |
| When use feature flags? | When you want to deploy code separately from releasing functionality to users. |
| Why are DB migrations risky during rolling deployments? | Old and new code run at the same time, so schema changes must be backward compatible. |
| What is shadow deployment? | Mirroring real traffic to a new version without returning its response to users. |

Are you ready for the next section?

