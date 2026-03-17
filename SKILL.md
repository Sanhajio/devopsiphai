---
name: devopsiphai
description: Audit and suggest improvements to the DevOps side of a project. Trigger whenever the user mentions auditing, reviewing, or improving their DevOps setup, CI/CD pipelines, Dockerfiles, IaC (Terraform, Helm, Nix), observability/monitoring, security, secrets management, architecture, FinOps, authentication, onboarding, or overall project health. Also trigger for "review my infra", "check my pipelines", "audit my project", "what's wrong with my devops", "is my project claude-onboardable", "review my repo structure", "give me an ARC score", "devopsiphai my project", or any request to assess the operational or architectural health of a codebase.
---

# devopsiphai

## Purpose

This skill audits the operational health of a project that is already in production.
It answers five questions that matter when you are running a live product:

1. **Can I onboard easily?**
   Can a new developer (or an AI agent) clone the repo, start the project,
   and contribute without asking anyone for anything?

2. **Can I deploy safely?**
   Can I ship a new version to staging and production without being afraid
   of breaking either environment? Are migrations automated? Is the same
   artifact promoted rather than rebuilt?

3. **Do I know what is running where?**
   After a deploy, can I tell exactly what version is live, when it was deployed,
   and what migration state the database is in — without digging through CI logs?

4. **Can I see what is happening?**
   After deploying, do I have metrics, error tracking, and analytics to know
   whether the deploy went well and whether users are behaving as expected?

5. **Can I recover if something goes wrong?**
   If a deploy causes errors that are reported, can I roll back the application,
   the database, and the frontend independently — and do I have a runbook for each?

The ARC score is the answer to these five questions expressed as a grade.

---

## ARC Framework

| Pillar | Answers | Covers |
|---|---|---|
| **Automation** | "Can I deploy safely?" + "Do I know what is running where?" | CI/CD pipelines, artifact promotion, migration automation, version tracking, pre-commit hooks, security scanning |
| **Reporting** | "Can I see what is happening?" | Observability, metrics, error tracking, alerting, dashboards, analytics, audit logs |
| **Control** | "Can I onboard easily?" + "Can I recover?" | Onboarding, secrets management, RBAC, backup/rollback strategy, IaC reproducibility, git auditability |

Grades: A (enforced) → B (present, partially enforced) → C (present, not enforced) → D (partially present) → F (absent)
One grade per pillar + one overall ARC grade. Scoring logic in `arc-scoring.md`.

---

## Phases

| Phase | File | Answers |
|---|---|---|
| 1 | `preliminary-audit.md` | Factual map of the project — sections 1.1–1.17, no judgement |
| 2 | `domain-audit.md` | Deep domain audits (CI/CD, containers, IaC, security, observability, onboarding) |
| 3 | `arc-scoring.md` | ARC grades + findings — directly answers the five questions |
| 4 | `workflow.md` | Can I onboard easily? — concrete checks derived from Phase 1 |
| 5 | `delivery.md` | Can I deploy safely + recover? — concrete checks derived from Phase 1 |
| 6 | `todogen.md` | Ordered TODO derived from all FAIL/PARTIAL checks and CRITICAL/HIGH/MEDIUM findings |

---

## Entrypoints

| User intent | Action |
|---|---|
| Full audit | Phases 1 → 2 → 3 → 4 → 5 → 6 |
| Exploration only / "preliminary audit" | `preliminary-audit.md` only |
| Specific section / "just check secrets" | Jump to relevant section in `preliminary-audit.md` |
| Domain audit only / "check my CI" | `domain-audit.md`, relevant domain only |
| ARC score only | `arc-scoring.md` (requires prior audit output) |
| Workflow checks only | `workflow.md` (requires Phase 1 output) |
| Delivery checks only | `delivery.md` (requires Phase 1 output) |
| Generate TODO only | `todogen.md` (requires Phases 3–5 output) |

---

## Runtime

- Read `config.yaml` first — apply skip_sections, skip_domains, arc_weights, output directory
- **Phase 1**: spawn one subagent per section (1.1 through 1.17) in parallel, skipping any in skip_sections
- Each subagent receives the project path and executes only its assigned section
- Collect all Phase 1 outputs and assemble in order before proceeding
- **Phase 2**: do not load until Phase 1 is complete — spawn one subagent per domain in parallel
- **Phase 3**: do not load until Phase 2 is complete — apply arc_weights from config
- **Phase 4**: do not load until Phase 3 is complete — every check derives from Phase 1 findings
- **Phase 5**: do not load until Phase 4 is complete — every check derives from Phase 1 and Phase 2 findings
- **Phase 6**: do not load until Phases 3, 4, and 5 are complete — derives entirely from prior outputs, no hardcoded content

---

## Output & Delivery

- On every run, create a timestamped directory: `/tmp/devopsiphai/<YYYY-MM-DD_HHMMSS>/`
- Write the full audit report to: `/tmp/devopsiphai/<YYYY-MM-DD_HHMMSS>/<project-name>-audit-<YYYY-MM-DD_HHMMSS>.md`
- Write the TODO separately to: `/tmp/devopsiphai/<YYYY-MM-DD_HHMMSS>/TODO.md`
- Read `post-run.md` after completing each run

---

## Rules

- Phase 1 is factual only — no suggestions, no judgement, no ✅/⚠️/❌
- Run all Phase 1 sections silently, dump full report when all sections complete
- Phases 4 and 5 produce PASS/FAIL/PARTIAL checks only — no free-form suggestions
- Findings and ARC grades are reserved for Phase 3
- TODO is reserved for Phase 6 — derived entirely from prior phase outputs, no invented content
- High-level recommendations only — no code snippets unless explicitly asked
- Load reference files in `domain-audit.md` only when that domain is actively being audited
- Never invent findings — if something is absent, state it as absent
