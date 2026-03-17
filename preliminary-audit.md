---
name: devopsiphai-explore
description: Run a factual, explorational preliminary audit (Phase 1) of a project.
Trigger when the user says "preliminary audit", "explore this project", "Phase 1 audit",
or "what is this project". No suggestions or judgement — facts only.
---

# Phase 1 — Preliminary Audit

Map the project as it exists. This phase is strictly factual and explorational.

## Rules

- **Factual only.** Describe what exists, not whether it is good or bad.
- **No suggestions.** Do not recommend improvements, alternatives, or fixes.
- **No judgement.** Do not use evaluative language: "unfortunately", "ideally",
  "should", "missing", "lacks", "poor", "good", "properly", or similar.
- **No ✅ / ⚠️ / ❌ icons.** State presence or absence neutrally.
- If something is absent, state it as absent — not as a problem.
- Findings, scoring, and recommendations are strictly reserved for `arc-scoring.md`.

## Execution

Run all sections silently in parallel — one subagent per section (1.1 through 1.15).
Each subagent receives the project path and executes only its assigned section.
Collect all outputs internally and dump the full report once all sections are complete.

---

### 1.1 Repository Structure

Investigate:
- Single repo or multiple repos? Look for: workspaces, turborepo, nx, lerna, multiple
  `package.json` / `pyproject.toml`, a `services/` or `apps/` top-level structure
- If multiple repos are provided or detected, run this section once per repo
- Map the top-level directories and what each one represents
- Is the project modular? Look for feature folders, bounded contexts, domain-driven
  structure vs. flat layout
- Run the following git commands per repo:
  ```bash
  git remote -v
  git log --format='%ae' | sort -u
  git shortlog -sne --all
  git log -1 --format='%H %ai %s'
  git tag --sort=-creatordate | head -5
  git branch -r
  git log --oneline | wc -l
  cat .github/CODEOWNERS 2>/dev/null || echo "No CODEOWNERS"
  ```

Output (single repo):
```
#### 1.1 Repository Structure
- **Remote**: `github.com/org/repo-name`
- **Latest commit**: `abc1234` — YYYY-MM-DD — "commit message"
- **Total commits**: N
- **Branches**: list all remote branches
- **Last 5 tags**: list or "None"
- **Contributors** (by commit count): list name + email + count
- **CODEOWNERS**: Present / Not detected
- **Type**: Monorepo / Single service
- **Top-level layout**:
  - `dir/` — description
  _(list all top-level directories with one-line description)_
- **Modularity**: Flat / Feature folders / Bounded contexts / Domain-driven
  _Description: ..._
- **Notable**: _anything structurally unusual_
```

Output (multi-repo — repeat block per repo, then summary):
```
#### 1.1 Repository Structure — `github.com/org/repo-name`
- **Remote**: `github.com/org/repo-name`
- **Latest commit**: `abc1234` — YYYY-MM-DD — "commit message"
- **Total commits**: N
- **Branches**: list all remote branches
- **Last 5 tags**: list or "None"
- **Contributors** (by commit count): list name + email + count
- **CODEOWNERS**: Present / Not detected
- **Role**: what this repo is responsible for
- **Top-level layout**:
  - `dir/` — description
  _(list all top-level directories)_
- **Modularity**: Flat / Feature folders / Bounded contexts / Domain-driven
  _Description: ..._
- **Notable**: _anything structurally unusual_

--- (repeat for each repo) ---

#### 1.1 Multi-repo Summary
- **Repos audited**: list repo names
- **Shared libraries**: Dedicated repo / Copy-pasted across repos / None detected
- **Versioning convention**: Semantic versioning / No convention detected
- **Cross-repo CI coordination**: Present / Not detected
```

---

### 1.2 Languages, Package Managers & Stack

Investigate:
- Which languages are used? Look for file extensions, lockfiles, manifests
- Which package managers?
  - Python: `requirements.txt`, `pyproject.toml` + Poetry, `conda`, `Pipfile`
  - JS/TS: `package.json` + check for `pnpm-lock.yaml`, `yarn.lock`, `package-lock.json`
  - Rust: `Cargo.toml` / `Cargo.lock`
  - Java/Kotlin: `pom.xml` (Maven), `build.gradle` (Gradle)
  - Go: `go.mod`
  - Ruby: `Gemfile`
- Which frameworks and runtimes?
  - Backend: FastAPI, Django, Express, NestJS, Spring Boot, Gin, Axum, Rails
  - Frontend: React, Next.js, Vue, Nuxt, SvelteKit, Angular
  - Mobile: React Native, Flutter, Expo, Swift (iOS), Kotlin (Android)
  - Deployment: Vercel, Netlify, Fly.io, Railway, Docker, Kubernetes
- Note which service uses which language in multi-language repos

Output:
```
#### 1.2 Languages, Package Managers & Stack
- **Languages**: list with versions
- **Package managers**:
  - Language: tool (lockfile name)
- **Backend stack**: framework + key libraries
- **Frontend stack**: framework + key libraries
- **Mobile**: None detected / framework
- **Deployment target**: list targets per service
- **Per-service breakdown**:
  - `service/` — language, package manager, framework
```

---

### 1.3 Architecture

Investigate:
- What is the high-level architecture pattern?
  - 3-tier (frontend / API / DB)
  - Event-driven (queues: RabbitMQ, Kafka, SQS, BullMQ, Celery, webhooks)
  - Microservices / monolith / modular monolith
  - Serverless (Lambda, Cloud Functions, Vercel Edge, Supabase Edge Functions)
- Is there an API gateway or reverse proxy? (nginx, Caddy, Kong, Traefik)
- Is there a cache layer? Where and what is cached?
- Is there an event bus or message queue?
- Estimate feature velocity from git history:
  ```bash
  # Average time from feature branch creation to merge into staging/main
  git log --merges --format="%ai %s" | head -50
  # Count releases per month
  git tag --sort=-creatordate --format="%(creatordate:short)" | head -20
  ```

Output:
```
#### 1.3 Architecture
- **Pattern**: label — description of how services interact
- **Services identified**:
  - `service/` — role, framework
- **API gateway / proxy**: name / None detected
- **Cache layer**:
  - location — what is cached, TTL if detectable
- **Event bus / queue**: name / None detected
- **Feature velocity**:
  - Avg merges to staging/main per week: N
  - Release cadence: every N days (estimated from tags)
  - Branch lifetime: estimated avg days from open to merge
```

---

### 1.4 Authentication & Identity

Investigate:
- What auth backend is in use?
  - Supabase Auth, Clerk, Firebase Auth, Keycloak, Auth0, NextAuth, Passport.js,
    custom JWT, session-based
- Where is token validation happening?
- Is RBAC present? Where is it implemented?
- How are auth secrets provided to the application?

Output:
```
#### 1.4 Authentication & Identity
- **Auth backend**: name + version
- **Token validation location**: middleware / per-route / client-side only / not found
- **RBAC**: Present at location / Not detected
- **Auth secret source**: environment variable / Vault / AWS Secrets Manager / hardcoded
- **Notable**: _e.g. multiple auth systems, auth only on some routes_
```

---

### 1.5 Database & Storage

Investigate:
- What databases are in use?
- Is Supabase used? Self-hosted or cloud?
- Are there S3 buckets or object storage?
  - Public or private? Versioning? Lifecycle rules?
- Are database migrations version-controlled?

Output:
```
#### 1.5 Database & Storage
- **Databases**: name + version / hosting
- **Supabase**: Cloud-hosted / Self-hosted / Not used
- **Object storage**:
  - Provider: name / None detected
  - Bucket visibility: Public / Private
  - Versioning: Enabled / Disabled / Not configured
  - Lifecycle rules: Defined / Not defined
- **Migrations**: tool + location / Raw SQL + location / None detected
- **Notable**: _anything unusual_
```

---

### 1.6 Secrets & Security Posture

Investigate:
- Grep for hardcoded secrets in source files, config files, `.env` committed to git,
  Docker Compose, Helm values, CI pipeline definitions
  - Patterns: `password=`, `secret=`, `api_key=`, `token=`, connection strings,
    `-----BEGIN`, `AKIA` (AWS key prefix)
- Is `.env` in `.gitignore`? Is there a `.env.example`?
- Is a secrets manager in use?
- Are any `*.pem`, `*.key`, `*.p12`, `*.pfx` files tracked in git?

Output:
```
#### 1.6 Secrets & Security Posture
- **Hardcoded secrets detected**: None / Found at: file:line (describe what type)
- **`.env` in `.gitignore`**: Present / Absent
- **`.env.example`**: Present / Absent
- **Secrets manager**: name / None detected
- **Private key files in git**: None / Found: list paths
- **Notable**: _e.g. credentials committed in specific commit hash_
```

---

### 1.7 Logging, Tracing & Observability

Investigate:
- What logging backend is in use?
- Is structured logging used?
- Is distributed tracing instrumented? Which libraries, where initialized?
- Is there an OpenTelemetry Collector?
- Are trace IDs correlated with logs?
- Are Prometheus metrics exported?
- Is there user action audit logging? Which system captures it?
  (core DB, Supabase audit log, third-party like Mixpanel/Segment, none)
- Are alerting rules defined in the codebase?
  (Prometheus rules files, PagerDuty config, Grafana alert JSON, none)
- Are dashboards version-controlled in the repo?
  (Grafana JSON, Datadog dashboard JSON, none)

Output:
```
#### 1.7 Logging, Tracing & Observability
- **Logging backend**: name / stdout only
- **Log format**: JSON structured / Plaintext / Mixed
- **Log levels**: list levels in use / Inconsistent / Not found
- **Tracing**:
  - Library: name / None detected
  - Initialized at: location
  - Exporter: name / Not configured
- **OTel Collector**: Found at path / Not detected
  - Exporters configured: list
- **Trace ↔ Log correlation**: trace_id in log fields / Not detected
- **Metrics endpoint**: path (Prometheus) / StatsD / None detected
- **User action audit logging**:
  - Present / Absent
  - System: core DB / Supabase audit / third-party (name) / None
  - What is logged: list event types if detectable
- **Alerting rules in codebase**: Found at path / None detected
- **Dashboards in repo**: Found at path / None detected
- **Notable**: _anything unusual_
```

---

### 1.8 FinOps & Hosting

Investigate:
- What is the hosting model per service?
- List all SaaS dependencies
- Which cloud providers and which specific services?
- Are there cost configuration signals?
- Is infrastructure defined in IaC?

Output:
```
#### 1.8 FinOps & Hosting
- **Hosting model**:
  - service: provider (type)
- **SaaS dependencies**: list all detected
- **Cloud provider**: name
  - Services in use: list
  - Regions: list
- **Cost configuration**:
  - S3 lifecycle rules: Defined / Not defined / N/A
  - Auto-scaling caps: Defined / Not defined
  - Budget alerts: Defined / Not defined
  - Resource limits (CPU/memory): Defined / Not defined
- **IaC coverage**: Full / Partial (list what is and isn't covered) / None detected
- **Notable**: _e.g. multiple regions, cross-cloud dependencies_
```

---

### 1.9 Onboarding

Investigate:
- Is there a README? What does it cover?
- Is there a single command to start the project?
- Are environment variables documented?
- Does a local dev environment exist?
  - Docker path: docker-compose up, make dev, etc.
  - Raw host path: install scripts, Makefile, nix develop, manual steps
- Does local mirror production?
- Is there a testing section in the docs?
  - How to run unit tests locally
  - How to run e2e tests locally
  - Is test data seeding documented?
- What requires live external credentials vs what can run offline?
- **Claude-onboardability**: Can an AI agent clone and spin up without human intervention?

Output:
```
#### 1.9 Onboarding
- **README**: Present (covers: list topics) / Absent
- **Docker setup**:
  - Single command: command / Multiple steps required / Not documented
  - Volumes or prerequisites required: list / None
- **Raw host setup**:
  - Single command: command / Multiple steps required / Not documented
  - Prerequisites documented: list / Not documented
- **Env vars documented**: tool/file + count documented vs total required
- **Local mirrors production**: Yes / Partial (list differences) / No
- **External services requiring live credentials**: list all
- **Testing**:
  - Unit tests: how to run / Not documented
  - Integration tests: how to run / Not documented
  - E2E tests: how to run / Not documented
  - Test data seeding: documented / Not documented
- **Claude-onboardable**: Yes / Partial / No
  _Blockers: list specific blockers_
- **Self-hostable analysis**:
  - Can be self-hosted: list services
  - Requires SaaS: list services with reason
```

---

### 1.10 Git Auditability

Investigate:
- List explicitly what configuration is and isn't in git:
  - CI/CD workflows, Dockerfiles, IaC, Helm charts, app config, dashboards,
    migration scripts, seed scripts, alerting rules
- Are secrets encrypted in git or externalized?
- Is `.gitignore` present and appropriate?
- Are there environment-specific config files?
- Are deployment configs reproducible or implicit (hardcoded in CI secrets)?
- Are migration runs part of CI or manual?
- Check git history for accidental secret commits

Output:
```
#### 1.10 Git Auditability
- **In git**:
  - Present: list what is versioned
  - Absent: list what is not versioned
- **Secrets handling**: SOPS / git-crypt / Externalized (name) / Plain .env / Hardcoded
  - Committed secrets found: None / list file + commit hash
- **`.gitignore`**: Present / Absent
  - Gaps detected: list any missing entries
- **Environment-specific configs**: list files / Single config only / None
- **Deploy configs reproducible**: Yes / No — list what is hardcoded in CI secrets
- **Migration runs**: Automated in CI / Manual process / Not documented
- **Git history anomalies**: None / list (force push, scrub attempt, credential commit)
```

---

### 1.11 Backup & Rollback Strategy

Investigate:
- Is there a database backup strategy?
- Are S3 buckets versioned?
- What is the deployment rollback mechanism?
- Is there a documented restore process?

Output:
```
#### 1.11 Backup & Rollback
- **DB backup**: tool/service + frequency / Manual only / None detected
- **S3 versioning**: Enabled / Disabled / Not applicable
- **Deployment rollback**: mechanism / None detected
- **Restore runbook**: Found at path / Not detected
- **Notable**: _anything unusual_
```

---

### 1.12 Git Workflow

Investigate:
- Infer branching strategy from branch names, merge commit messages, PR titles
  - Gitflow (main + develop + feature/* + release/* + hotfix/*)
  - Trunk-based (main only, short-lived feature branches)
  - Ticket-driven (TEL-xxx, JIRA-xxx prefix branches)
  - GitHub flow (main + feature branches + PRs)
- Is the workflow documented? (CONTRIBUTING.md, docs/git-workflow.md, README section)
- What is the merge strategy? (squash, merge commit, rebase)
  ```bash
  git log --oneline --merges | head -20
  git log --oneline | grep -E "^[a-f0-9]+ Merge" | head -10
  ```
- What is the release process? (tags, changelog, release branch, none)
- Are there branch protection rules detectable from CI config?

Output:
```
#### 1.12 Git Workflow
- **Branching strategy**: name — description of inferred pattern
- **Documented**: Found at path / Not documented — inferred from history
- **Branch naming convention**: pattern (e.g. TEL-xxx-description, feature/xxx)
- **Merge strategy**: Squash / Merge commit / Rebase / Mixed
- **Default branch**: name
- **Release process**: Tags + changelog / Tags only / Manual / Not detected
- **Branch protection**: Detectable in CI config / Not detectable
- **Notable**: _e.g. staging is HEAD target, not main_
```

---

### 1.13 Testing

Investigate:
- What test frameworks are present per service?
  - Python: pytest, unittest, nose
  - JS/TS: Vitest, Jest, Mocha, Cypress, Playwright
- What types of tests exist?
  - Unit, integration, e2e, snapshot, contract
- Is there a coverage configuration? What threshold?
- Are tests run in CI?
- Docker image security scanning:
  - Trivy, Snyk, Grype, Docker Scout, none
- SAST tools:
  - Python: Bandit, Semgrep, SonarQube, CodeQL
  - JS/TS: ESLint security plugins, Semgrep, SonarQube, CodeQL
- Secret scanning:
  - detect-secrets, gitleaks, trufflehog, GitHub secret scanning, none

Output:
```
#### 1.13 Testing
- **Test frameworks**:
  - service: framework + version
- **Test types present**:
  - Unit: Present at path / Absent
  - Integration: Present at path / Absent
  - E2E: Present at path / Absent
  - Snapshot: Present / Absent
- **Coverage**: Threshold configured (N%) / Configured, no threshold / Not configured
- **Tests in CI**: Yes (list jobs) / No
- **Docker image scanning**: tool / None detected
- **SAST**: tool + scope / None detected
- **Secret scanning**: tool / None detected
```

---

### 1.14 AI Usage

Investigate:
- Is there a CLAUDE.md or .claude/ directory?
- Are there Copilot, Cursor, or Codeium config files? (.cursorrules, .copilot/)
- Are LLM API calls made in CI pipelines?
- Are there AI automation scripts in the repo?
- Are there AI usage guidelines documented anywhere?
- What AI/LLM SDKs are in dependencies?

Output:
```
#### 1.14 AI Usage
- **CLAUDE.md**: Present at path / Absent
- **Other AI config**: list files found / None detected
- **LLM SDKs in dependencies**: list (provider + version)
- **LLM calls in CI**: Present (describe) / Not detected
- **AI automation scripts**: Found at path / None detected
- **AI usage guidelines**: Found at path / None detected
- **Notable**: _e.g. multiple LLM providers, agent framework in use_
```

---

### 1.15 Code Quality & Patterns

Investigate per service:
- What paradigm is used? (OOP, functional, mixed)
- What architectural pattern? (MVC, hexagonal, clean arch, repository pattern, none recognizable)
- For 3-tier: do the folders actually map to the pattern?
  - Is there a clear controllers / services / models / repositories split?
  - Or is logic mixed across files?
- For Next.js frontend:
  - What state management stores exist? (Zustand, Redux, Jotai, Context)
  - What does each store contain? Are models and functions split?
  - Are API calls co-located with components or in a service layer?
- File size distribution:
  ```bash
  # Find files over 300 lines
  find . -name "*.py" -o -name "*.ts" -o -name "*.tsx" | \
    xargs wc -l 2>/dev/null | sort -rn | head -20
  ```
- Pre-commit hooks:
  - `.pre-commit-config.yaml`, `husky`, `lefthook` — what hooks are configured?
  - Are hooks enforced in CI (can't be bypassed)?
- Formatters:
  - Python: black, ruff format, autopep8, isort — config location
  - JS/TS: prettier — config location (.prettierrc, package.json)
  - Are formatters enforced in CI or pre-commit only?

Output:
```
#### 1.15 Code Quality & Patterns

**Backend (`service/`)**
- **Paradigm**: OOP / Functional / Mixed
- **Pattern**: MVC / Hexagonal / Clean arch / Repository / None recognizable
- **Folder → pattern mapping**:
  - `folder/` → role in pattern
  _(map all key directories to their architectural role)_
- **Logic separation**: Controllers / services / models split / Mixed in single files
- **Largest files** (over 300 lines):
  - `path/file.py` — N lines
- **Pre-commit hooks**: list hooks configured / None detected
  - Enforced in CI: Yes / No
- **Formatters**: list tools + config location / None detected
  - Enforced in CI: Yes / Pre-commit only / No

**Frontend (`service/`)**
- **Paradigm**: Component-based / OOP / Mixed
- **Pattern**: Feature folders / Atomic design / Flat / None recognizable
- **State management**:
  - Store: name — contents (what state it holds)
  _(list all stores)_
- **API call location**: Dedicated service layer / Co-located with components / Mixed
- **Models / functions split**: Separate files / Co-located / Not applicable
- **Largest files** (over 300 lines):
  - `path/file.tsx` — N lines
- **Pre-commit hooks**: list hooks / None detected
  - Enforced in CI: Yes / No
- **Formatters**: list tools + config location / None detected
  - Enforced in CI: Yes / Pre-commit only / No
```


### 1.16 Questionable Architecture Decisions
 
Investigate for patterns that are technically functional but architecturally unusual,
expensive, or likely to cause friction. State findings factually — do not judge,
but flag them clearly for Phase 3 to evaluate.
 
Look for:
- Docker volumes used to persist build artifacts or package dependencies
  (e.g. `node_modules`, `.next`, `__pycache__` as named or external volumes)
  — these prevent fresh startup and make the environment non-reproducible
- External Docker volumes declared as `external: true` that must exist before `docker compose up`
- Services that could run in Docker but are instead hitting external SaaS
  (e.g. Supabase cloud used for local dev when self-hosted Supabase could be used)
- SaaS dependencies used for local dev that have a self-hostable equivalent:
  | SaaS | Self-hostable alternative |
  |---|---|
  | Supabase cloud | `supabase start` (Supabase CLI) or Docker Compose stack |
  | Auth0 / Clerk | Keycloak, Ory Kratos, self-hosted Supabase Auth |
  | Pinecone | Qdrant, Weaviate, pgvector |
  | Algolia | Meilisearch, Typesense |
  | SendGrid / Mailgun | Mailpit, Mailhog (local dev), Postal (self-hosted) |
  | Datadog / New Relic | Grafana + Prometheus + Loki (self-hosted) |
  | Sentry (SaaS) | Self-hosted Sentry, GlitchTip |
  | Pusher | Soketi, self-hosted Ably |
  | Firebase | Supabase, PocketBase |
- Frontend analytics using paid or privacy-invasive SaaS where open-source alternatives exist:
  | SaaS | Self-hostable alternative |
  |---|---|
  | Google Analytics | Plausible, Umami, Matomo |
  | Mixpanel | PostHog (self-hosted) |
  | Amplitude | PostHog (self-hosted) |
  | Hotjar | OpenReplay (self-hosted) |
- Any service running in Docker Compose in production but not locally
- Configuration that is environment-specific but not expressed in version-controlled files
- Choices that break reproducibility: hardcoded IPs, absolute paths, machine-specific config
 
Output:
```
#### 1.16 Questionable Architecture Decisions
- **External volumes blocking startup**: list volumes + which services depend on them / None detected
- **SaaS used for local dev with self-hostable alternative**:
  - service: current SaaS — alternative: tool
  _(list each)_
- **Services in Docker for prod but not local**: list / None detected
- **Frontend analytics**:
  - Detected: list tools
  - Self-hostable alternatives available: list / N/A
- **Other reproducibility concerns**: list / None detected
```
 
---
 
### 1.17 AI Workflow Configuration
 
Investigate for AI coding assistant configuration files:
 
- `CLAUDE.md` — Claude Code project instructions
- `AGENTS.md` — OpenAI Codex / general agent instructions
- `.cursorrules` — Cursor IDE rules (legacy format)
- `.cursor/rules/` — Cursor IDE rules (directory format)
- `.github/copilot-instructions.md` — GitHub Copilot workspace instructions
- `.aider.conf.yml` or `CONVENTIONS.md` — Aider instructions
- Any file named `AI_GUIDELINES.md`, `LLM_CONTEXT.md`, or similar
 
For each found, note: location, approximate size, what it covers.
 
Output:
```
#### 1.17 AI Workflow Configuration
- **CLAUDE.md**: Found at path (N lines, covers: list topics) / Absent
- **AGENTS.md**: Found at path / Absent
- **.cursorrules**: Found at path / Absent
- **.cursor/rules/**: Found at path / Absent
- **Copilot instructions**: Found at path / Absent
- **Other AI config**: list / None detected
```

