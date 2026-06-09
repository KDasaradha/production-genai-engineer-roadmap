# CI/CD And GitHub Actions

## 1. Problem Statement

CI/CD automates the process of validating, building, securing, publishing, and deploying applications.

Without CI/CD:

- developers manually run tests
- builds differ by machine
- releases are inconsistent
- security checks are skipped
- deployment approvals are informal
- rollback is harder

Real-world analogy: CI/CD is an automated assembly line for software delivery.

## 2. Beginner Explanation

| Topic | Notes |
| --- | --- |
| CI | Continuous Integration validates code changes with linting, tests, and checks. |
| CD | Continuous Delivery/Deployment prepares or performs release into environments. |
| Workflow | A GitHub Actions automation file. |
| Job | A set of steps running on a runner. |
| Step | One command/action inside a job. |
| Artifact | Output produced by a workflow, such as reports or build files. |
| Cache | Stored dependency/build data to speed up workflows. |
| Environment | GitHub deployment target with protection rules and secrets. |
| Use When | Every serious repo needs repeatable validation and delivery. |
| Interview Answer | CI/CD turns code changes into validated, versioned, deployable artifacts through automated workflows and environment promotion gates. |

## 3. Modern Pipeline

```text
Pull Request
  |
  v
Lint
  |
  v
Tests
  |
  v
Security Scan
  |
  v
Build
  |
  v
SBOM
  |
  v
Sign
  |
  v
Push Registry
  |
  v
Deploy
  |
  v
Verify
```

## 4. Repository Layout

Recommended app repository:

```text
app-repo/
  app/
    api/
    core/
    models/
    services/
  tests/
  Dockerfile
  pyproject.toml
  .pre-commit-config.yaml
  .github/
    workflows/
      pr-validation.yml
      build-publish.yml
      deploy-dev.yml
      promote-staging.yml
      promote-prod.yml
```

Recommended GitOps repository:

```text
gitops-repo/
  apps/
    fastapi/
      base/
      overlays/
        dev/
        staging/
        prod/
  clusters/
    dev/
    staging/
    prod/
```

## 5. Branch Strategy

| Branch Type | Purpose | Rules |
| --- | --- | --- |
| `main` | Production-ready source | protected, PR required, checks required |
| feature branch | New work | short-lived, merged by PR |
| hotfix branch | Urgent production fix | reviewed and promoted carefully |
| release tag | Immutable release point | used for traceability |

Recommended:

```text
feature/* -> PR -> main -> build image -> promote dev -> staging -> prod
```

## 6. Environment Strategy

| Environment | Purpose | Protection |
| --- | --- | --- |
| Dev | Fast feedback and integration | automatic deployment allowed |
| Staging | Production-like validation | approval recommended |
| Prod | User-facing system | required approval, restricted secrets, audit trail |

Promotion flow:

```text
Merge to main
  |
  v
Build image once
  |
  v
Deploy to dev
  |
  v
Promote same image to staging
  |
  v
Promote same image to production
```

Important rule: do not rebuild for every environment. Build once, promote the same artifact.

## 7. GitHub Actions Workflow Architecture

### 1. PR Validation

Runs on every pull request:

```text
PR Created
  |
  +-- Ruff
  +-- Black check
  +-- Mypy
  +-- Unit tests
  +-- Bandit
  +-- Semgrep
  +-- Gitleaks
  +-- Trivy filesystem/IaC scan
```

Purpose: block bad code before merge.

### 2. Build And Publish

Runs after merge to `main`:

```text
Checkout code
  |
  v
Build Docker image
  |
  v
Generate SBOM
  |
  v
Scan image
  |
  v
Sign image
  |
  v
Push to registry
```

### 3. Dev Deployment

Runs automatically after build:

```text
New signed image
  |
  v
Update dev GitOps manifest
  |
  v
ArgoCD syncs dev
```

### 4. Integration Test Workflow

Runs after dev deployment:

```text
Smoke tests
API tests
Database migration check
Security smoke checks
```

### 5. Staging Promotion

Runs after approval:

```text
Approve staging
  |
  v
Promote same image digest
  |
  v
Run staging validation
```

### 6. Security Testing Stage

Runs before production:

```text
DAST
Dependency review
Image scan result verification
SBOM verification
Signature verification
Policy checks
```

### 7. Production Promotion

Runs after manual approval:

```text
Approve production
  |
  v
Promote same image digest
  |
  v
Canary or rolling rollout
  |
  v
Post-deploy verification
```

### 8. Canary Rollout

Gradually shifts traffic:

```text
5 percent
  |
25 percent
  |
50 percent
  |
100 percent
```

Rollback if error rate, latency, or business metrics degrade.

### 9. Nightly Security Workflow

Runs on schedule:

```text
Dependency scan
Container scan
IaC scan
Secrets scan
SBOM freshness check
```

### 10. Dependency Update Workflow

Use Dependabot or Renovate for:

```text
Python dependencies
Docker base images
GitHub Actions versions
Kubernetes chart versions
```

## 8. GitHub Secrets Strategy

Do not store long-lived cloud keys if you can use OIDC.

Recommended:

```text
GitHub Actions
  |
  v
OIDC federation
  |
  v
Cloud IAM role
  |
  v
Temporary credentials
```

Use GitHub environment secrets only for environment-specific values that belong in CI/CD.

## 9. GitHub Environment Protection

| Environment | Protection |
| --- | --- |
| Dev | automatic or lightweight approval |
| Staging | reviewer approval, staging-only secrets |
| Prod | required reviewers, restricted branches, deployment history, audit trail |

## 10. Release Artifacts

Each release should produce:

- Docker image digest
- SBOM
- vulnerability scan report
- test report
- signed artifact
- Git tag
- changelog or release notes
- deployment record

## 11. Example FastAPI CI Workflow

```yaml
name: pr-validation

on:
  pull_request:

jobs:
  validate:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: "3.12"
      - run: pip install -r requirements-dev.txt
      - run: ruff check .
      - run: mypy app
      - run: pytest
      - run: bandit -r app
```

## 12. Common Mistakes

| Mistake | Fix |
| --- | --- |
| Deploying untested code | Require PR validation before merge |
| Rebuilding per environment | Build once and promote the same image digest |
| Long-lived cloud credentials in GitHub | Use OIDC and temporary credentials |
| No environment approvals | Use GitHub protected environments |
| Security scans only on release | Scan on PR, build, nightly, and before production |
| Ignoring workflow artifacts | Store test, scan, SBOM, and signing evidence |

## 13. Interview Preparation

| Question | Model Answer |
| --- | --- |
| What is CI? | CI automatically validates code changes through linting, tests, and security checks before merge. |
| What is CD? | CD automates release preparation or deployment through build, artifact publishing, promotion, and verification. |
| Why build once and promote? | It guarantees dev, staging, and prod use the same artifact, improving reproducibility and auditability. |
| What are GitHub environments? | Protected deployment targets with secrets, approvals, and deployment history. |
| What should run on a PR? | Linting, unit tests, type checks, SAST, dependency scanning, secret scanning, IaC scanning, and relevant container checks. |

Are you ready for the next section?

