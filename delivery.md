# Phase 5 — Delivery Enhancement

Read Phase 1 and Phase 3 outputs in full before generating any checks.
Every check must derive from what was actually detected — do not assume any specific
CI provider, registry, deploy target, or version tracking mechanism.

---

## Rules

- Every check is a yes/no question derived from prior phase findings
- Adapt to the detected stack: Helm values, Kustomize overlays, SSM parameters,
  versions.json, or any other version tracking mechanism are all valid
- Never invent a check for a tool that was not detected
- FAIL and PARTIAL checks become TODO items in Phase 6

---

## 5.1 Artifact Identity

For each build artifact detected in Phase 1 (Docker image, Lambda zip, static bundle, etc.):

- [ ] Is the artifact tagged with the commit SHA at build time?
- [ ] Is the artifact tagged with a semver version at release time?
- [ ] Is the artifact tagged with anything other than a mutable tag (`:latest`, `main`, `staging`)?
  (mutable-only tagging makes rollback impossible — flag if detected)

For version tracking — check whether the stack has a mechanism that records
what is deployed where. This could be any of:
- A committed file (`versions.json`, `deployed.yaml`, `Chart.lock`, etc.)
- Helm release values with image tag pinned
- Kustomize overlays with image digest pinned
- SSM/Secrets Manager parameter storing current version
- Git tag convention (`deployed/staging-v1.1.1`)
- CI environment variable set on deploy
- None detected

Checks:
- [ ] Is there a version tracking mechanism for staging?
- [ ] Is there a version tracking mechanism for production?
- [ ] Does the version tracking record the artifact version deployed?
- [ ] Does the version tracking record the migration checkpoint deployed?
- [ ] Is the version tracking updated automatically by CI on every successful deploy?

---

## 5.2 Pipeline Correctness

For each CI/CD pipeline detected in Phase 2 (domain audit — CI/CD):

- [ ] Does the staging pipeline follow this order: build → test → migrate → deploy → post-deploy smoke test?
  (deploying before testing means broken code reaches the environment — flag if the order differs)
- [ ] Does the production pipeline promote the artifact that passed staging
  instead of rebuilding from source?
- [ ] Is it impossible (by pipeline design) to deploy to production without
  the artifact having passed staging?
- [ ] Are staging and production pipeline definitions structurally identical?
  (same steps, different secrets and targets)
- [ ] Are reusable pipeline components used to avoid duplicating definitions?
  (GitHub reusable workflows, GitLab includes, Jenkins shared libraries, etc.)
- [ ] Does the pipeline stop and fail if migrations fail?
- [ ] Does the pipeline notify on deploy failure?
  (Slack webhook, email, PagerDuty, whatever notification mechanism is in use)
- [ ] Does the pipeline notify on successful production deploy?

---

## 5.3 Migration Automation

For the migration tool detected in Phase 1 section 1.5:

- [ ] Are migrations run automatically in the staging deploy pipeline
  before the new application version starts?
- [ ] Are migrations run automatically in the production deploy pipeline
  before the new application version starts?
- [ ] Is there a rollback step that reverts migrations if the deploy fails?
- [ ] Is the migration checkpoint recorded in the version tracking mechanism
  after each successful deploy?

---

## 5.4 Rollback

**Application rollback**
- [ ] Is there a named recipe (just, make, script) that redeploys a specific
  artifact version to a named environment?
- [ ] Does that recipe read the target version from the version tracking mechanism
  or accept an explicit version argument?
- [ ] Does that recipe redeploy without rebuilding (pulls from registry/artifact store)?
- [ ] Does that recipe update the version tracking after a successful rollback?

**Database rollback — safe path**
For the database tool detected in Phase 1 section 1.5:
- [ ] Does a down migration exist for every up migration?
- [ ] Is there a named recipe that runs down migrations to a specific checkpoint?
- [ ] Is the safe rollback condition documented?
  (safe only when no data was written after the migration ran)

**Database rollback — unsafe path**
- [ ] Is point-in-time restore (PITR) or equivalent enabled?
  (Supabase PITR, RDS automated backups, MongoDB Atlas PITR, pg_basebackup, etc.)
- [ ] Is the PITR retention window documented?
- [ ] Is there a runbook that describes the restore procedure,
  including the explicit data loss window?

**Frontend rollback**
(only if a frontend hosting platform was detected in Phase 1 section 1.8)
- [ ] Does the hosting platform support instant rollback to a previous deployment?
- [ ] Is frontend rollback documented as independent from backend rollback?
- [ ] If a frontend change requires a coordinated backend change,
  is that dependency noted so rollbacks can be coordinated?

---

## 5.5 Infrastructure Codification

For each infrastructure component detected in Phase 1 section 1.8:

- [ ] Is there a codified definition for this component?
  Valid forms: Terraform, Pulumi, CDK, CloudFormation, Helm chart, Kustomize overlay,
  Docker Compose, shell script, Python script, Ansible playbook, Nix module
- [ ] Is that definition version-controlled in the repo?
- [ ] Can the infrastructure be reproduced from the repo alone, starting from zero?

For each manually provisioned component detected (flagged in Phase 2 IaC audit):
- [ ] Is there at minimum a documented runbook for reproducing it manually?

---

## Output Format

```
## Phase 5 — Delivery Enhancement

### 5.1 Artifact Identity
- <artifact> commit SHA tag: PASS / FAIL
- <artifact> semver tag: PASS / FAIL
- Mutable-only tag detected: Yes (flag) / No
- Version tracking mechanism: <name> / None detected
- Version tracking — staging: PASS / FAIL / N/A
- Version tracking — production: PASS / FAIL / N/A
- Version tracking — migration checkpoint: PASS / FAIL / N/A
- Version tracking — updated by CI: PASS / FAIL / N/A

### 5.2 Pipeline Correctness
- Staging pipeline order (build → test → migrate → deploy → smoke): PASS / FAIL / PARTIAL
- Production promotes staging artifact: PASS / FAIL
- Production cannot bypass staging: PASS / FAIL
- Staging/production pipeline symmetry: PASS / FAIL / PARTIAL
- Reusable pipeline components: PASS / FAIL
- Pipeline stops on migration failure: PASS / FAIL
- Failure notification: PASS / FAIL
- Production deploy notification: PASS / FAIL

### 5.3 Migration Automation
- Migrations in staging pipeline: PASS / FAIL
- Migrations in production pipeline: PASS / FAIL
- Migration rollback step: PASS / FAIL
- Migration checkpoint tracked: PASS / FAIL

### 5.4 Rollback
- Application rollback recipe: PASS / FAIL
- Down migrations present: PASS / FAIL / PARTIAL
- Safe rollback recipe: PASS / FAIL
- PITR / backup enabled: PASS / FAIL / UNKNOWN
- PITR retention documented: PASS / FAIL / N/A
- Restore runbook: PASS / FAIL
- Frontend rollback available: PASS / FAIL / N/A
- Coordinated rollback documented: PASS / FAIL / N/A

### 5.5 Infrastructure Codification
- <component>: codified (<tool>) / manual / manual + runbook
_(one line per component detected in Phase 1 section 1.8)_
- Reproducible from repo: Yes / Partial / No
```
