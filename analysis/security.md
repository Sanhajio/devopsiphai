# Security Domain Audit

Read this file when auditing security posture. Always run on every project.
Feeds into the **Control** ARC pillar (secrets, RBAC) and **Automation** (scanning).

---

## What to investigate

### Secret Scanning
- Run grep patterns across entire repo:
  ```bash
  grep -rn "AKIA[0-9A-Z]{16}" .                        # AWS access key
  grep -rn "-----BEGIN.*PRIVATE KEY" .                  # private keys
  grep -rn "password\s*=\s*['\"][^'\"]{6,}" .           # hardcoded passwords
  grep -rn "secret\s*=\s*['\"][^'\"]{6,}" .             # hardcoded secrets
  grep -rn "api_key\s*=\s*['\"][^'\"]{6,}" .            # API keys
  grep -rn "sk-[a-zA-Z0-9]{20,}" .                      # OpenAI-style keys
  grep -rn "postgresql://.*:.*@" .                       # DB connection strings
  ```
- Are any `.env` files tracked in git?
- Are any `*.pem`, `*.key`, `*.p12`, `*.pfx` tracked?
- Check git log for commits that added then removed credentials

### Secrets Management
- Is a secrets manager used? (Vault, AWS Secrets Manager, Doppler, SOPS, age)
- Are secrets different per environment (dev/staging/prod)?
- Are secrets rotated? (look for rotation scripts, docs, or CI jobs)
- Is there a `.env.example` covering all required vars?

### Docker Image Security
- Is the base image pinned to a digest or specific version (not `latest`)?
- Does the container run as non-root?
- Is a minimal base image used? (alpine, distroless, slim)
- Is there a `.dockerignore` excluding `.git`, secrets, local configs?
- Multi-stage build to exclude build tools from final image?
- Are image scans configured? (Trivy, Snyk, Grype, Docker Scout)

### SAST & Code Scanning
- Python: Bandit, Semgrep, SonarQube, CodeQL configured?
- JS/TS: ESLint security plugins, Semgrep, SonarQube, CodeQL configured?
- Are scans run in CI or only locally?

### Secret Scanning Hooks
- detect-secrets, gitleaks, trufflehog in pre-commit or CI?
- GitHub secret scanning enabled (detectable from security config)?

### Dependency Security
- Are dependencies pinned to exact versions?
- Are there dependency audit jobs in CI? (pip-audit, npm audit, Safety)
- Are there known CVEs in pinned versions? (check top-level deps)

### Authentication Security
- Are JWT secrets strong and externalized?
- Is token expiry configured?
- Is there rate limiting on auth endpoints?
- Are auth bypass paths detectable?

### Network & API Security
- Is CORS configured? Is it locked to specific origins or wildcard?
- Are there rate limiting middleware configured?
- Is HTTPS enforced?
- Are internal services exposed unnecessarily?

---

## Output Format

```
## Security Audit

### Secret Scanning Results
- Hardcoded secrets found: None / list type + file:line
- .env files in git: None / list paths
- Private key files in git: None / list paths
- Credentials in git history: None / commit hash + description

### Secrets Management
- Secrets manager: name / None
- Per-environment secrets: Yes / No
- .env.example coverage: N of total required vars documented

### Docker Image Security
- Base image pinned: Yes / No — current value
- Runs as non-root: Yes / No
- Minimal base image: Yes (type) / No
- .dockerignore present: Yes / No — gaps: list
- Multi-stage build: Yes / No
- Image scanning: tool / None

### SAST
- Tools configured: list / None
- Runs in CI: Yes / No / Not applicable

### Secret Scanning Hooks
- Tools configured: list / None
- Runs in CI: Yes / No

### Dependency Security
- Versions pinned: Yes / Partial / No
- Audit in CI: Yes / No
- Notable CVEs: list if detectable / None detected

### Auth Security
- JWT externalized: Yes / No
- Token expiry configured: Yes / No / Not detected
- Rate limiting: Present at location / Not detected
- CORS: Locked to origins / Wildcard / Not configured

### Gaps Detected
- list factual gaps
```
