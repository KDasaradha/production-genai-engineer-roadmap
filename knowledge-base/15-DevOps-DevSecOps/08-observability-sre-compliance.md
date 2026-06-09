# Observability, SRE, And Compliance

## 1. Problem Statement

Production systems fail. Observability helps you understand what happened. SRE helps you manage reliability. Compliance helps you prove controls exist and are followed.

Without these:

- incidents take longer
- teams guess instead of diagnose
- outages repeat
- audits become painful
- reliability goals are unclear

## 2. Beginner Explanation

| Topic | Notes |
| --- | --- |
| Observability | Ability to understand system behavior from outputs like metrics, logs, and traces. |
| Metrics | Numeric time-series data such as latency, error rate, CPU, memory. |
| Logs | Event records from application and infrastructure. |
| Traces | End-to-end request flow across services. |
| SRE | Site Reliability Engineering applies software engineering to operations and reliability. |
| SLI | Service Level Indicator, a reliability measurement. |
| SLO | Service Level Objective, a target for an SLI. |
| SLA | Service Level Agreement, external promise to users/customers. |
| Error budget | Allowed unreliability before releases should slow down. |
| Interview Answer | Observability tells you what production is doing; SRE turns that data into reliability goals, alerts, incident response, and improvement loops. |

## 3. Runtime Observability Stack

```text
FastAPI
   |
   v
OpenTelemetry
   |
   +-- Metrics
   +-- Logs
   +-- Traces
   |
   v
Prometheus + Loki + Tempo/Tracing Backend
   |
   v
Grafana Dashboards
   |
   v
Alertmanager
```

Recommended stack:

| Tool | Purpose |
| --- | --- |
| OpenTelemetry | standard instrumentation for traces, metrics, and logs |
| Prometheus | metrics collection and querying |
| Grafana | dashboards and visualization |
| Loki | log aggregation |
| Alertmanager | alert routing |

## 4. Three Pillars

```text
Metrics
Logs
Traces
```

Metrics answer: "How much and how often?"

Logs answer: "What event happened?"

Traces answer: "Where did the request spend time?"

## 5. Observability Beyond Logs

Minimum FastAPI signals:

- request count
- request latency
- error rate
- dependency latency
- database query latency
- queue depth
- worker failure rate
- pod restarts
- memory/CPU usage
- saturation of PostgreSQL and Redis
- deployment version labels

Do not rely only on application logs.

## 6. Observability After Deployment

After every deployment, verify:

- pods are healthy
- readiness checks pass
- error rate is normal
- latency is normal
- logs do not show repeated exceptions
- database connection count is safe
- queues are not backing up
- business-critical flows work

## 7. Incident Response

Incident response lifecycle:

```text
Detect
  |
  v
Triage
  |
  v
Mitigate
  |
  v
Communicate
  |
  v
Resolve
  |
  v
Postmortem
  |
  v
Prevent recurrence
```

Tools and practices:

- SIEM
- audit logs
- threat hunting
- runbooks
- on-call rotation
- postmortems
- escalation paths
- compliance reporting

## 8. Backup And Disaster Recovery

Definitions:

| Term | Meaning |
| --- | --- |
| Backup | Copy of data used for restore |
| Disaster recovery | Plan for recovering from major failure |
| RPO | Recovery Point Objective: maximum acceptable data loss |
| RTO | Recovery Time Objective: maximum acceptable downtime |

Example:

```text
RPO = 15 minutes
RTO = 1 hour
```

Production requirements:

- automated backups
- restore testing
- documented recovery steps
- database snapshots
- object storage backup strategy
- secrets and config recovery
- region/zone failure planning where needed

## 9. SRE Concepts

```text
SLI -> measured reliability
SLO -> target reliability
SLA -> external commitment
Error budget -> allowed failure budget
```

Example:

```text
SLI: successful request rate
SLO: 99.9 percent successful requests per month
SLA: contract promise to customer
Error budget: 0.1 percent monthly failure allowance
```

## 10. Compliance Readiness

### SOC2

SOC2 focuses on trust service criteria such as security, availability, confidentiality, processing integrity, and privacy.

Evidence examples:

- access controls
- deployment approvals
- audit logs
- incident response records
- vulnerability management
- backup evidence
- change management

### ISO27001

ISO27001 focuses on an Information Security Management System.

Evidence examples:

- risk register
- policies
- controls
- internal audits
- access reviews
- incident management
- asset inventory

## 11. Continuous Compliance

Continuous compliance means controls produce evidence during normal work.

Examples:

```text
GitHub PR approvals
GitHub environment approvals
CI/CD scan artifacts
Image signatures
SBOMs
Audit logs
Incident tickets
Access review records
Backup restore tests
```

## 12. Runtime Security

Runtime security watches actual running systems.

Tools:

```text
Falco
eBPF monitoring
SIEM
Kubernetes audit logs
```

Detect:

- shell opened inside container
- unexpected process
- suspicious file access
- privilege escalation
- network anomaly
- unexpected outbound connection

## 13. Common Mistakes

| Mistake | Fix |
| --- | --- |
| Logs only, no metrics/traces | Add OpenTelemetry, Prometheus, Grafana, and tracing |
| Alerts too noisy | Alert on user impact and actionable symptoms |
| No postmortems | Document causes, timeline, and prevention |
| Backups never tested | Schedule restore tests |
| Compliance evidence gathered manually at audit time | Generate evidence continuously through workflows |
| No SLOs | Define reliability targets and error budgets |

## 14. Interview Preparation

| Question | Model Answer |
| --- | --- |
| Metrics vs logs vs traces? | Metrics show numeric trends, logs show events, traces show request paths across services. |
| What is an SLO? | A target reliability objective for a measured SLI, such as 99.9 percent successful requests. |
| RPO vs RTO? | RPO is acceptable data loss. RTO is acceptable downtime. |
| What is incident response? | A structured process to detect, triage, mitigate, communicate, resolve, and learn from production incidents. |
| What is continuous compliance? | Producing audit evidence automatically as part of normal engineering workflows. |

Are you ready for the next section?

