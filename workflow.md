# Phase 4 — Workflow Enhancement
 
Read Phase 1 output in full before generating any checks.
Every check must derive from what was actually detected — do not assume any specific
tool, language, database, or environment variable mechanism.
 
---
 
## Rules
 
- Every check is a yes/no question derived from Phase 1 findings
- If Phase 1 found Python → check for `.python-version`. If it found Node → check for `.nvmrc`. If neither → skip.
- If Phase 1 found Postgres → check for a dump/load script for that DB. If it found MongoDB → check for mongodump. If it found Supabase → check for Supabase CLI export.
- If Phase 1 found `.env` files → check for `.env.example`. If it found AWS SSM → check for SSM parameter documentation. If it found Vault → check for Vault path documentation.
- Never invent a check for a tool that was not detected
- FAIL and PARTIAL checks become TODO items in Phase 6
 
---
 
## 4.1 Runtime Version Lockdown
 
For each runtime detected in Phase 1 section 1.2, check whether its version is locked in a file:
 
| Runtime | Lock file to check |
|---|---|
| Node.js | `.nvmrc` or `.node-version` |
| Python | `.python-version` (pyenv) |
| Ruby | `.ruby-version` |
| Java | `.java-version` or `.sdkmanrc` |
| Go | `go.mod` (already pins version) |
| Rust | `rust-toolchain.toml` |
| Multiple runtimes | `.tool-versions` (asdf) |
| Nix | `flake.nix` devShell |
 
Additional check: do version lock files match the versions used in Dockerfiles and CI?
 
---
 
## 4.2 Developer Environment
 
The goal is an environment that is:
- Scoped to the individual developer — not shared, not owned by anyone else
- Startable without asking another person for anything
- Able to run without credentials owned by external parties, wherever possible
 
For each infrastructure component detected in Phase 1 sections 1.3, 1.5, and 1.8, check:
 
- [ ] Is there a developer-scoped instance of this component?
  (e.g. local DB container prefixed with dev name, local queue, local object storage)
- [ ] Is that instance startable with a single command without prior manual setup?
 
For the database detected in Phase 1 section 1.5, check:
- [ ] Is there a script to export/dump the staging database to a local file?
  (use the tool appropriate for the detected DB: pg_dump, mongodump, mysqldump, Supabase CLI export, etc.)
- [ ] Is there a script to import/load that dump into the developer-local database instance?
 
For environment variable management detected in Phase 1 section 1.6 and 1.9, check:
- [ ] Is there a script that generates env files from local infrastructure outputs?
  (reads values from running containers, writes to the env file format used by the stack)
- [ ] If a secrets provider is used (AWS SSM, Vault, Doppler, AWS Secrets Manager):
  is there a script or recipe that pulls the required secrets for a given environment?
- [ ] Is there a documented env file template for each service listing every required variable?
  (`.env.example`, `dotenv` docs, `config/settings.py` with comments, etc. — whatever fits the stack)
- [ ] Are variables that cannot be self-hosted explicitly marked as requiring external credentials,
  with a note on where to obtain them?
 
For local setup overall:
- [ ] Is there a single command that performs full setup from a fresh clone?
  (start infrastructure, generate env files, run migrations, seed data, start dev servers)
- [ ] Does that command work idempotently (safe to run twice without side effects)?
 
---
 
## 4.3 Testing Layers
 
For each test framework detected in Phase 1 section 1.13, check:
 
**Unit tests**
- [ ] Are unit tests runnable with a single command without any running infrastructure?
- [ ] Is there a coverage threshold configured?
- [ ] Do unit tests run in CI on every PR?
 
**Backend integration / e2e**
(only if a backend service was detected)
- [ ] Are there tests that hit the actual running backend server with a real database?
- [ ] Do those tests read their configuration (server URL, DB URL) from the environment?
- [ ] Can those tests run against a local stack by setting one env var?
- [ ] Can those tests run against a remote stack (staging, beta) by changing that env var?
- [ ] Is there test data isolation? (isolated schema, fixtures that reset between runs, test DB prefix)
- [ ] Do backend e2e tests run in CI after each staging deploy?
 
**Frontend e2e**
(only if a frontend was detected)
- [ ] Are there tests that drive a real browser or make real HTTP calls to the backend?
- [ ] Do those tests read the backend URL from an environment variable?
- [ ] Can those tests target local, staging, or production by changing that variable?
- [ ] Do frontend e2e tests run in CI after each staging deploy?
 
---
 
## 4.4 Task Runner
 
For the task runner detected in Phase 1 section 1.9 (justfile, Makefile, package.json scripts, etc.),
or if none was detected, suggest one appropriate for the stack.
 
Check whether each of these operations has a named recipe:
 
- [ ] Full setup from fresh clone
- [ ] Start dev servers
- [ ] Run all tests
- [ ] Run unit tests only
- [ ] Run backend e2e tests against a configurable target
- [ ] Run frontend e2e tests against a configurable target
- [ ] Run database migrations against a named environment
- [ ] Export/dump the database from a remote environment
- [ ] Import/load a database dump into the local environment
- [ ] Seed the local database
- [ ] Generate env files from local infrastructure
- [ ] Check health of all local services
- [ ] Rollback to a previous version (see Phase 5)
 
---
 
## Output Format
 
```
## Phase 4 — Workflow Enhancement
 
### 4.1 Runtime Version Lockdown
- <runtime>: PASS — found at <path> / FAIL — not found / PARTIAL — <description>
_(one line per runtime detected in Phase 1 section 1.2)_
 
### 4.2 Developer Environment
- Developer-scoped <component>: PASS / FAIL / PARTIAL — <description>
  _(one line per infrastructure component detected in Phase 1)_
- <DB type> dump script: PASS / FAIL / PARTIAL
- <DB type> load script: PASS / FAIL / PARTIAL
- Env file generation script: PASS / FAIL / PARTIAL
- Secrets provider pull script: PASS / FAIL / PARTIAL / N/A
- Env template (<.env.example or equivalent>): PASS / FAIL / PARTIAL
- External credential documentation: PASS / FAIL / PARTIAL
- Single-command setup: PASS / FAIL / PARTIAL — <blocker description>
- Idempotent setup: PASS / FAIL / PARTIAL
 
### 4.3 Testing Layers
- Unit tests — single command: PASS / FAIL
- Unit tests — coverage threshold: PASS / FAIL
- Unit tests — run in CI: PASS / FAIL
- Backend e2e — hits real server: PASS / FAIL / N/A
- Backend e2e — configurable target: PASS / FAIL / N/A
- Backend e2e — test data isolation: PASS / FAIL / N/A
- Backend e2e — run in CI after staging: PASS / FAIL / N/A
- Frontend e2e — real browser/HTTP: PASS / FAIL / N/A
- Frontend e2e — configurable target: PASS / FAIL / N/A
- Frontend e2e — run in CI after staging: PASS / FAIL / N/A
 
### 4.4 Task Runner
- setup: PASS / FAIL
- dev: PASS / FAIL
- test: PASS / FAIL
_(one line per recipe above)_
```
 
---
 
## 4.5 Local-First Stack
 
The principle: anything that can run in Docker should run locally.
Self-hosted is preferred over SaaS for local dev wherever a viable alternative exists.
 
For each SaaS dependency detected in Phase 1 sections 1.8 and 1.16:
 
- [ ] Does a self-hostable alternative exist for local dev?
  (derive from the table in section 1.16 — do not invent alternatives)
- [ ] If yes: is that alternative running in the local Docker Compose stack?
- [ ] Is there a single Docker Compose file (or set of compose files) that starts
  the complete local stack — application services + all infrastructure dependencies?
  (DB, cache, queue, object storage, search, auth, email, analytics — whatever was detected)
- [ ] Are there any services that run in the production Docker Compose
  but are absent from the local Docker Compose?
- [ ] For SaaS that cannot be self-hosted (OpenAI, Anthropic, Stripe, Clerk, etc.):
  is the minimum required credential set documented so a developer knows
  exactly which external accounts are needed and nothing more?
- [ ] For SaaS that cannot be self-hosted: is there a cheaper tier or usage cap
  that could be applied for local/staging use?
  (e.g. Pinecone free tier, OpenAI spending limits, Sentry free plan)
 
---
 
## 4.6 E2E Test Complexity
 
E2E tests are significantly harder to set up and maintain than unit tests.
The audit should reflect this honestly.
 
- [ ] Is there a documented, working procedure to run e2e tests locally
  from a fresh clone (not just "run `pytest` or `playwright test`")?
  This includes: starting the full stack, seeding test data, configuring
  the test runner to point at the local stack, and cleaning up after.
- [ ] Are e2e tests isolated from dev data?
  (separate test DB, test schema prefix, fixtures that reset between runs)
- [ ] Is the e2e test suite fast enough to run on every PR?
  (flag if suite takes > 10 minutes — suggest splitting smoke tests from full suite)
- [ ] Is there a smoke test subset that runs in CI on every PR
  and a full e2e suite that runs only on staging deploy?
- [ ] Are flaky tests tracked and quarantined?
  (a flaky e2e suite that blocks CI is worse than no e2e suite)
 
---
 
## 4.7 AI Workflow Files
 
Based on Phase 1 section 1.17 findings:
 
- [ ] Is there a `CLAUDE.md` at the repo root covering:
  project overview, how to run the project, architecture summary,
  conventions, and common tasks?
- [ ] Is there an `AGENTS.md` or equivalent for other AI coding assistants
  used by the team?
- [ ] If Cursor is used (detected via `.cursorrules` or Makefile opening Cursor):
  is there a `.cursor/rules/` directory with project-specific rules?
- [ ] Do AI config files reflect the current project structure?
  (outdated AI instructions are worse than none — flag if files exist but
  reference paths, commands, or tools that no longer exist)

