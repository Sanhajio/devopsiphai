# Containers Domain Audit

Read this file when auditing Docker/container setup. Feeds into **Automation** ARC pillar.

---

## What to investigate

### Base Image
- Is the base image pinned to a digest or specific version (not `latest`)?
- Is a minimal base used? (alpine, distroless, slim, bookworm-slim)
- Is the base image from a trusted registry?

### Build Quality
- Are layers ordered from least-to-most frequently changing?
  (system deps → app deps → source code)
- Is multi-stage build used to exclude build tools from final image?
- Is `--no-cache-dir` or equivalent used for package managers?
- Is there a `.dockerignore` excluding: `.git`, `node_modules`, `.env`, secrets?

### Runtime Security
- Does the container run as a non-root user? (`USER` directive present)
- Are there read-only filesystem directives?
- Are capabilities dropped?

### Health & Reliability
- Are health checks defined? (`HEALTHCHECK` or Compose `healthcheck`)
- Are resource limits (CPU/memory) defined in Compose or orchestrator?
- Are restart policies defined?

### Compose / Orchestration
- Are environment variables sourced from `.env` or secrets, not hardcoded?
- Are service dependencies declared with `depends_on` and health conditions?
- Is there a production-specific override (`docker-compose.prod.yml`)?
- Are named volumes used for persistent data?
- Are there external volume dependencies requiring manual creation?

### Image Scanning
- Is Trivy, Snyk, Grype, or Docker Scout configured?
- Does scanning run in CI before push to registry?
- Is there a severity threshold that fails the build?

### Registry
- Which registry is used? (ECR, Docker Hub, GHCR, GCR)
- Are images tagged with commit SHA or version (not just `latest`)?
- Is there an image retention/cleanup policy?

---

## Output Format

```
## Containers Audit

### Dockerfiles Found
- `path/Dockerfile` — service: description
_(list all Dockerfiles)_

### Base Images
- service: `image:tag` — pinned to digest / version tag / latest

### Build Quality
- Layer ordering: Correct / Incorrect — description
- Multi-stage build: Yes / No — per service
- .dockerignore: Present / Absent — gaps: list

### Runtime Security
- Non-root user: Yes / No — per service
- Read-only filesystem: Yes / No
- Capabilities dropped: Yes / No / Not configured

### Health & Reliability
- Health checks: Present / Absent — per service
- Resource limits: Defined / Not defined — per service
- Restart policy: Defined / Not defined

### Compose Configuration
- Env vars source: .env file / hardcoded / secrets mount
- depends_on with health: Yes / No
- Prod override file: Present / Absent
- External volume dependencies: list / None

### Image Scanning
- Tool: name / None detected
- Runs in CI: Yes / No
- Severity threshold: configured / not configured

### Registry
- Provider: name
- Image tagging: commit SHA / version / latest only
- Retention policy: Defined / Not defined

### Gaps Detected
- list factual gaps
```
