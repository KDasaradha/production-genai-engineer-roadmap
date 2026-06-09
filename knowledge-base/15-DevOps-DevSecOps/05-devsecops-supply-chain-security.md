# DevSecOps And Supply Chain Security

## 1. Problem Statement

Modern applications depend on many third-party packages, container images, CI/CD workflows, infrastructure templates, registries, secrets, and runtime platforms.

The security question is no longer only:

```text
Is my code secure?
```

It is:

```text
Can I prove what code was built, what dependencies were included, who built it, where it was stored, whether it was signed, and whether production is allowed to run it?
```

Without supply chain security:

- vulnerable packages enter production
- secrets leak into Git
- containers ship with known CVEs
- images can be tampered with
- teams cannot audit releases
- compliance evidence is weak

## 2. Beginner Explanation

| Topic | Notes |
| --- | --- |
| DevSecOps | Security embedded into development, CI/CD, artifact handling, deployment, runtime, and compliance. |
| SAST | Static Application Security Testing scans source code. |
| SCA | Software Composition Analysis scans dependencies. |
| Secret scanning | Detects credentials in source, commits, and configs. |
| Container scanning | Scans images and OS packages for vulnerabilities. |
| IaC scanning | Scans Terraform, Kubernetes YAML, and cloud configs. |
| SBOM | Software Bill of Materials lists components in an artifact. |
| Signing | Cryptographically proves artifact integrity and origin. |
| SLSA | Supply-chain maturity framework for software artifacts. |
| Interview Answer | DevSecOps shifts security left and also secures the build, artifact, deployment, runtime, and compliance lifecycle. |

## 3. Modern DevSecOps Lifecycle

```text
Developer Workstation
  |
  v
Pre-commit hooks
  |
  v
Pull request security
  |
  v
Build security
  |
  v
Artifact signing and SBOM
  |
  v
Registry controls
  |
  v
Policy-based deployment approval
  |
  v
Runtime security
  |
  v
Incident response and compliance
```

## 4. Developer Workstation Security

Before code reaches Git:

```text
Developer
  |
  +-- Ruff
  +-- Black
  +-- Mypy
  +-- Bandit
  +-- Detect-Secrets
  +-- Gitleaks
  +-- Pre-commit
```

Tool purpose:

| Tool | Purpose |
| --- | --- |
| Ruff | linting and fast Python code checks |
| Black | formatting |
| Mypy | static type checking |
| Bandit | Python security scanning |
| Detect-Secrets | secrets baseline and detection |
| Gitleaks | secret detection in Git history and files |
| pre-commit | run checks before commits |

## 5. Pull Request Security

Every PR should trigger:

```text
Lint
Unit tests
SAST
Dependency scan
Secret scan
IaC scan
Container scan
```

Security scanners:

| Tool | Use |
| --- | --- |
| Semgrep | SAST across Python, JS, Java, and more |
| Trivy | dependency, container, filesystem, and IaC scanning |
| Checkov | Terraform and Kubernetes security |
| OWASP Dependency Check | dependency vulnerability scanning |

## 6. Source Code Security

| Language | Tools |
| --- | --- |
| Python | Bandit, Semgrep |
| Java | SpotBugs, Semgrep |
| JavaScript/TypeScript | ESLint security rules, Semgrep |

Common FastAPI issues to scan for:

- missing authentication
- insecure CORS
- SQL injection patterns
- unsafe deserialization
- hardcoded secrets
- weak JWT validation
- overly verbose errors
- missing input validation

## 7. Secrets Security

Never keep secrets in:

```text
.env committed to Git
GitHub repository variables
Docker image layers
Kubernetes manifests
application source code
logs
```

Better flow:

```text
Vault / AWS Secrets Manager
        |
        v
External Secrets Operator
        |
        v
Kubernetes Secret
        |
        v
Application Pod
```

Recommended tools:

- HashiCorp Vault
- AWS Secrets Manager
- External Secrets Operator
- Gitleaks
- Detect-Secrets

## 8. Dependency Security

Dependency scanning protects against vulnerable packages.

Tools:

```text
Trivy
Dependabot
OWASP Dependency Check
```

Best practices:

- pin dependencies
- use lock files
- automate dependency updates
- review transitive dependencies
- block critical vulnerabilities where practical
- create an exception process for accepted risk

## 9. Container Security

Container risks:

- vulnerable base images
- running as root
- unnecessary packages
- leaked secrets in image layers
- unpinned base image tags
- no image scanning

Recommended:

```text
Use slim base images
Pin versions
Run as non-root
Scan with Trivy
Sign with Cosign
Push to private registry
Deploy by digest
```

## 10. Supply Chain Security

Build stage:

```text
Source
  |
  v
Docker Build
  |
  +-- Generate SBOM
  +-- Scan Image
  +-- Sign Image
  +-- Push Registry
```

Tools:

| Tool | Purpose |
| --- | --- |
| Syft | Generate SBOM |
| Cosign | Sign images and artifacts |
| Trivy | Scan filesystems, dependencies, containers, IaC |
| Sigstore | Ecosystem for keyless signing and verification |

## 11. SBOM

SBOM means Software Bill of Materials.

```text
Application
  |
  v
Dependencies
  |
  v
SBOM
```

Why it matters:

- vulnerability response
- compliance evidence
- dependency visibility
- release traceability

Interview answer: An SBOM lists the software components included in an artifact so teams can identify exposure when vulnerabilities are discovered.

## 12. Signing

Signing ensures image integrity and provenance.

```text
Image digest
  |
  v
Cosign signature
  |
  v
Policy allows deployment only if signature is valid
```

Interview answer: Signing proves that an artifact was produced by a trusted pipeline and was not modified before deployment.

## 13. SLSA

SLSA means Supply-chain Levels for Software Artifacts.

Target:

```text
SLSA Level 3+
```

The goal is stronger build provenance, tamper resistance, and artifact traceability.

## 14. Security Gates

Block immediately:

- detected secrets
- critical exploitable vulnerabilities
- unsigned production image
- missing SBOM
- failed tests
- policy violation
- deployment from untrusted branch

Warning only:

- low severity vulnerabilities
- non-critical dependency updates
- informational findings
- accepted risk with expiration

## 15. Registry Security

Registry controls:

- private repositories
- least-privilege push/pull access
- image scanning
- immutable tags
- image signing
- SBOM storage
- retention policies
- audit logs

## 16. Production Example

Enterprise FastAPI supply chain:

```text
GitHub PR
  |
  v
SAST + SCA + secret scan
  |
  v
Docker build
  |
  v
SBOM with Syft
  |
  v
Trivy image scan
  |
  v
Cosign signature
  |
  v
Push to ECR
  |
  v
Kyverno verifies signature before deployment
```

## 17. Common Mistakes

| Mistake | Fix |
| --- | --- |
| Treating DevSecOps as one scanner | Secure code, dependencies, secrets, images, IaC, artifacts, registry, deployment, and runtime |
| Not generating SBOMs | Generate SBOMs for every release artifact |
| Signing but not verifying | Enforce signature verification during deployment |
| Ignoring secrets in Docker layers | Never copy `.env` into images |
| Blocking every low severity issue | Use severity thresholds and risk exceptions |

## 18. Interview Preparation

| Question | Model Answer |
| --- | --- |
| SAST vs SCA? | SAST scans your source code for insecure patterns. SCA scans dependencies for known vulnerabilities and license/security risks. |
| Why use Gitleaks? | To detect secrets before they enter Git history or CI/CD artifacts. |
| What is an SBOM? | A list of software components inside an artifact, used for vulnerability response and compliance. |
| Why sign container images? | To prove artifact integrity and trusted build origin before deployment. |
| What should block production deployment? | Critical vulnerabilities, leaked secrets, unsigned images, missing SBOMs, failed tests, or policy violations. |

Are you ready for the next section?

