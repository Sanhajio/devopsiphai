# Onboarding Domain Audit

Read this file when auditing onboarding. Always run on every project.
Answers the question: **Can I onboard easily?**
Feeds into the **Control** ARC pillar.

---

## What to investigate

### Documentation Coverage
- Is there a root-level README?
- Does it cover: architecture overview, setup instructions, env vars, testing?
- Is there a CONTRIBUTING.md?
- Is there a dedicated docs/ directory?
- Are architecture decisions recorded? (ADRs, docs/architecture/)
- Does the README reference a task runner (just, make, etc.) for common operations?

### Docker Setup Path
- Is there a single command to start the full stack via Docker?
- Are there prerequisites that must exist before that command works?
  (external volumes, manually created networks, pre-existing containers)
- Does the compose file include ALL services needed for local dev?
  List what is included and what is missing:
  - DB, cache, queue, object storage, search, auth, email — whatever was detected
  - Or does it rely on external SaaS for services that could run locally?
- Is there a seed or fixture script to populate the DB with test data?
- Are external Docker volumes declared as `external: true`?
  (these block fresh startup — flag each one)

### Raw Host Setup Path
- Is there a documented raw host setup (no Docker required)?
- Is there a Makefile, justfile, shell script, or nix devShell?
- Are language version requirements locked in files?
  (.nvmrc, .python-version, .tool-versions, rust-toolchain.toml)
- Are system-level dependencies listed? (apt packages, brew formulas, etc.)
- Is there a single command to start each service individually?

### Environment Variables
- Total required env vars (count from app settings, config files, Pydantic models, etc.)
- Total documented in .env.example, dotenv docs, or equivalent
- Are variables grouped by service or concern?
- Are required vs optional vars distinguished?
- Are variables that cannot be self-hosted (Clerk, Stripe, OpenAI, etc.)
  explicitly marked with a note on where to obtain them?
- Is there a script that generates env files from local infrastructure outputs?
  (reads ports/keys from running containers, writes to .env)

### Testing Documentation
- Is there a documented procedure to run unit tests from a fresh clone?
- Is there a documented procedure to run integration/e2e tests from a fresh clone?
  (not just "run pytest" — full procedure: start stack, seed data, configure target, run, clean up)
- Is test data seeding documented?
- Are tests runnable without live external service credentials?

### Local-First Assessment
For each external SaaS dependency detected in Phase 1 sections 1.8 and 1.16:
- Can this dependency be replaced with a self-hosted alternative for local dev?
- Is that alternative already in the local Docker Compose stack?
- If not self-hostable: is the minimum required credential documented?

### Claude-Onboardability
Score these six checks against what was found:
1. Can the project be cloned and started with ≤ 3 commands?
2. Are all env vars documented with example values?
3. Are external service dependencies avoidable for basic local dev?
4. Does the Docker Compose include all required services (no external volumes blocking startup)?
5. Is there a raw host path documented?
6. Are tests runnable without external service credentials?

---

## Output Format

```
## Onboarding Audit

### Documentation
- Root README: Present (covers: list topics) / Absent
- CONTRIBUTING.md: Present / Absent
- docs/ directory: Present (contains: list) / Absent
- Architecture docs / ADRs: Present / Absent
- Task runner referenced: Yes / No

### Docker Setup
- Single command: `command` / Multiple manual steps / Not documented
- External volumes blocking startup: list volumes / None
- Prerequisites: list / None
- Services in compose: list included / Missing: list absent
- Local DB included: Yes / No — relies on: service name
- Seed script: Present at path / Absent

### Raw Host Setup
- Documented: Yes / No
- Language version files: list (.nvmrc, .python-version, etc.) / None
- System deps documented: Yes / No
- Single-command start per service: Yes / No

### Environment Variables
- Total required: N
- Total documented: N (N%)
- Grouping: By service / Flat / Not documented
- Required vs optional distinguished: Yes / No
- External credential guidance: Present / Absent
- Env generation script: Present at path / Absent

### Testing
- Unit tests: `command` to run / Not documented
- E2E tests: full procedure documented / Partially documented / Not documented
- Test data seeding: Documented / Not documented
- Offline testable: Yes / Partial (list what requires live credentials) / No

### Local-First Assessment
| Service | Self-hostable | In local compose | Min credential documented |
|---|---|---|---|
| name | Yes/No | Yes/No | Yes/No |

### Claude-Onboardability
1. ≤ 3 commands to start: Yes / No — blockers: list
2. All env vars documented: Yes / No — gap: N undocumented
3. External deps avoidable: Yes / Partial / No — list required live services
4. Docker Compose complete (no external volume blocks): Yes / No — list issues
5. Raw host path documented: Yes / No
6. Tests offline: Yes / Partial / No

**Claude-onboardable**: Yes / Partial / No

### Gaps Detected
- list factual gaps with specific file references
```
