# Phase 3 — ARC Scoring

This phase synthesizes findings from `preliminary-audit.md` and `domain-audit.md`
into a structured ARC score. This is where judgement, suggestions, and grades live.

---

## ARC Grade Scale

```
A  — Present and enforced (automated, no manual bypass possible)
B  — Present, partially enforced (some gaps, mostly automated)
C  — Present, not enforced (exists but can be bypassed or is inconsistent)
D  — Partially present, not enforced (fragmented, incomplete)
F  — Absent (not present at all)
```

---

## ARC Pillars & Domain Mapping

### Automation (A)
Covers: CI/CD pipelines, artifact promotion, version tracking, pre-commit hooks, formatters,
security testing automation, migration automation, test automation, image scanning in CI

Domains that feed this pillar:
- CI/CD (weight: 40%)
- Testing — automation aspects (weight: 30%)
- Containers — build automation (weight: 15%)
- Code Quality — pre-commit + formatters (weight: 15%)

### Reporting (R)
Covers: Observability, alerting rules, dashboards, audit logs, error tracking,
user action logging, Slack/email notifications

Domains that feed this pillar:
- Observability (weight: 50%)
- Logging & Tracing from preliminary audit (weight: 30%)
- User action audit trail (weight: 20%)

### Control (C)
Covers: Secrets management, RBAC, backup/rollback, error-triggered rollback,
IaC reproducibility, git auditability, deploy reproducibility, onboarding

Domains that feed this pillar:
- Security (weight: 30%)
- IaC (weight: 25%)
- Backup & Rollback from preliminary audit (weight: 20%)
- Git Auditability from preliminary audit (weight: 25%)

---

## Scoring Output Format

```
# ARC Score — <project-name>
# Date: <YYYY-MM-DD>

## Five Questions

| Question | Answer | Grade |
|---|---|---|
| Can I onboard easily? | Yes / Partial / No | X |
| Can I deploy safely? | Yes / Partial / No | X |
| Do I know what is running where? | Yes / Partial / No | X |
| Can I see what is happening? | Yes / Partial / No | X |
| Can I recover if something goes wrong? | Yes / Partial / No | X |

## Pillar Scores

| Pillar | Grade | Summary |
|---|---|---|
| Automation | X | one-line description |
| Reporting  | X | one-line description |
| Control    | X | one-line description |
| **Overall ARC** | **X** | one-line description |

---

## Automation — Grade X

### What is present
- list what exists

### What is enforced
- list what is actually enforced vs optional

### Gaps
**[CRITICAL | HIGH | MEDIUM | LOW]** — <title>
> <what is missing and why it matters in context of Automation>
> Suggestion: <high-level recommendation>

---

## Reporting — Grade X

### What is present
- list what exists

### What is enforced
- list what is actually enforced vs optional

### Gaps
**[CRITICAL | HIGH | MEDIUM | LOW]** — <title>
> <what is missing and why it matters in context of Reporting>
> Suggestion: <high-level recommendation>

---

## Control — Grade X

### What is present
- list what exists

### What is enforced
- list what is actually enforced vs optional

### Gaps
**[CRITICAL | HIGH | MEDIUM | LOW]** — <title>
> <what is missing and why it matters in context of Control>
> Suggestion: <high-level recommendation>

---

## Priority Actions

Ordered by severity and ARC pillar impact:

1. **[CRITICAL]** <action> — fixes X in Automation/Reporting/Control
2. **[HIGH]** <action> — fixes X in Automation/Reporting/Control
3. ...

---

## Onboarding Score

| Check | Result |
|---|---|
| Docker setup documented | Yes / Partial / No |
| Raw host setup documented | Yes / Partial / No |
| Env vars documented | N / total required |
| Single-command start | Yes / No |
| Tests runnable locally | Yes / Partial / No |
| Claude-onboardable | Yes / Partial / No |

**Onboarding Grade**: X
```

---

## Grading Criteria per Pillar

### Automation grades

**A** — All of:
- CI runs on every PR and merge to main/staging
- Deployment is fully automated, no manual steps
- Artifacts tagged with commit SHA and semver — never mutable-only tags
- Version tracking records what is deployed where (artifact + migration checkpoint)
- Production promotes staging artifact — never rebuilds from source
- Security scanning (SAST, image scan, secret scan) runs in CI
- Migrations run automatically in CI before deploy
- Pre-commit hooks enforced in CI (can't be bypassed)
- Formatters enforced in CI

**B** — Most of the above, with 1–2 gaps (e.g. no image scanning, or migration is semi-manual)

**C** — CI exists but: manual deploy steps required, or hooks only local, or no security scanning

**D** — CI exists but is minimal (only build, no test/security/deploy automation)

**F** — No CI/CD detected

---

### Reporting grades

**A** — All of:
- Structured logging (JSON) with consistent levels
- Distributed tracing with trace IDs in logs
- Alerting rules defined in codebase (not just UI)
- Dashboards version-controlled in repo
- User action audit trail present
- Error tracking with context (Sentry, Datadog, etc.)
- Product analytics present (PostHog, Mixpanel, Plausible, or equivalent)
- Error alerts configured to notify on production errors

**B** — Error tracking present, logging present, but missing dashboards-as-code or alerting rules in repo

**C** — Error tracking only (Sentry), no structured logs, no alerting rules in codebase

**D** — Stdout logging only, no error tracking, no tracing

**F** — No observability tooling detected

---

### Control grades

**A** — All of:
- No secrets in git, secrets manager in use for all environments
- IaC covers all infrastructure, reproducible from scratch
- Backup automated and restore tested/documented
- Deployment rollback automated with rollback recipe per environment
- Alerting or error tracking is configured to surface errors that would trigger a rollback decision
- Rollback runbook documents the trigger condition, the command, and the data loss window
- Git history clean, all configs in git, env-specific configs separated
- Single-command onboarding from fresh clone

**B** — Secrets externalized for production, but dev has gaps; IaC partial; backup present but untested

**C** — Secrets in .env only (not git), no IaC, manual backup, manual rollback documented

**D** — Secrets partially hardcoded, no backup strategy, rollback is manual re-deploy

**F** — Secrets hardcoded in git, no backup, no rollback strategy
