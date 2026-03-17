# Onboarding Domain Audit

Read this file when auditing onboarding. Always run on every project.
Feeds into **Automation** (setup scripts) and **Control** (reproducibility) ARC pillars.

---

## What to investigate

### Documentation Coverage
- Is there a root-level README?
- Does it cover: architecture overview, setup instructions, env vars, testing?
- Is there a CONTRIBUTING.md?
- Is there a dedicated docs/ directory?
- Are architecture decisions recorded? (ADRs, docs/architecture/)

### Docker Setup Path
- Is there a single command to start the full stack via Docker?
  (`docker-compose up`, `make dev`, etc.)
- Are there prerequisites? (external volumes, network creation, manual steps)
- Does the compose file include all services needed for local dev?
  - DB, cache, queue, all app services
  - Or does it rely on external SaaS (Supabase, Pinecone, etc.) for local dev?
- Is there a seed/fixture script to populate test data?

### Raw Host Setup Path
- Is there a documented raw host setup? (no Docker required)
- Is there a Makefile, shell script, or nix devShell for setup?
- Are language version requirements documented? (.nvmrc, .python-version, .tool-versions)
- Are system dependencies listed? (apt packages, brew dependencies)
- Is there a single command to start each service individually?

### Environment Variables
- Total required env vars (count from app settings/config)
- Total documented in .env.example or docs
- Are they grouped by service/feature?
- Are required vs optional vars distinguished?
- Are there instructions for obtaining external service credentials?

### Testing Documentation
- Is there a documented way to run unit tests?
- Is there a documented way to run integration tests?
- Is there a documented way to run e2e tests?
- Is there test data seeding documentation?
- Is there a test environment setup guide?

### Self-Hosted vs SaaS Recommendations
For each external SaaS dependency, note:
- Whether a self-hosted alternative exists for local dev
- Whether the project could run fully offline for dev
- Cost implications of current SaaS choices at scale

### Claude-Onboardability Assessment
Score against these specific checks:
1. Can the project be cloned and started with ≤ 3 commands?
2. Are all env vars documented with example values?
3. Are all external service dependencies avoidable for basic local dev?
4. Is there a working Docker Compose that starts all required services?
5. Is there a raw host path documented?
6. Are tests runnable without external service credentials?

---

## Output Format

```
## Onboarding Audit

### Documentation
- Root README: Present (covers: list) / Absent
- CONTRIBUTING.md: Present / Absent
- docs/ directory: Present (contains: list) / Absent
- Architecture docs / ADRs: Present / Absent

### Docker Setup
- Single command: `command` / Multiple steps / Not documented
- Prerequisites: list / None
- Services in compose: list all / Missing: list absent services
- Local DB included: Yes / No — relies on: service name
- Seed script: Present / Absent

### Raw Host Setup
- Documented: Yes / No
- Language version files: list (.nvmrc, .python-version) / None
- System deps documented: Yes / No
- Single-command start per service: Yes / No

### Environment Variables
- Total required: N
- Total documented: N
- Coverage: N%
- Grouped by service: Yes / No
- Required vs optional distinguished: Yes / No
- Credential acquisition guide: Present / Absent

### Testing
- Unit tests runnable: `command` / Not documented
- Integration tests runnable: `command` / Not documented
- E2E tests runnable: `command` / Not documented
- Test data seeding: Documented / Not documented
- Offline testable: Yes / Partial / No

### Self-Hosted Feasibility
| Service | Self-hostable for dev | Alternative | Effort |
|---|---|---|---|
| name | Yes/No/Partial | tool | Low/Medium/High |

### Claude-Onboardability
1. ≤ 3 commands to start: Yes / No — actual commands or blockers
2. All env vars documented: Yes / No — gap count
3. External deps avoidable: Yes / No — list required live services
4. Docker Compose complete: Yes / No — missing services
5. Raw host path: Yes / No
6. Tests offline: Yes / No

**Claude-onboardable**: Yes / Partial / No

### Gaps Detected
- list factual gaps with specific file references
```
