# CI/CD Domain Audit

Read this file when auditing CI/CD pipelines. Feeds into the **Automation** ARC pillar.

---

## What to investigate

### Triggers
- What events trigger each pipeline? (push, PR, tag, schedule, manual)
- Is there a single trigger that deploys both frontend and backend together?
  - Or do they deploy independently — note if one can drift from the other
- Does a DB migration trigger automatically on deploy?
- Does a Vercel/Netlify deploy trigger from the same pipeline or separately?
- Are there manual approval gates? Where?

### Idempotency & Parity
- Is staging → production promotion idempotent?
  - Meaning: does deploying the same commit to staging and then to prod
    produce identical results, or are there environment-specific manual steps?
- Are staging and production pipelines symmetrical?
  - Same steps, different secrets — or structurally different?
- Can the pipeline be re-run safely without side effects?

### Pipeline Structure
- Are pipelines DRY or duplicated across repos/workflows?
- Is there a clear stage separation: lint → test → build → deploy?
- Are build artifacts reused between stages or rebuilt per stage?
- Are pipeline definitions version-controlled and in the repo?

### Security in CI
- Are secrets injected via CI secrets store (GitHub Secrets, GitLab CI vars)?
- Are third-party actions/orbs pinned to a commit SHA?
- Is there a least-privilege principle on CI runner permissions?
- Are image builds scanned before push to registry?

### Migration Automation
- Are DB migrations run automatically as part of deploy?
- Is there a pre-deploy migration step with rollback on failure?
- Or are migrations run manually before/after deploy?

### Deployment Targets
- What deploys where? Map each job to its target
  - AWS App Runner, Vercel, Netlify, k8s, EC2, etc.
- Are service ARNs, URLs, or deploy keys hardcoded in workflow files
  or properly referenced from secrets?

### Notifications
- Is there a Slack/email notification on pipeline failure?
- Is there a notification on successful production deploy?

---

## Output Format

```
## CI/CD Audit

### Pipeline Inventory
- Workflow: `filename.yml` — trigger: event — deploys: target
_(list all workflows)_

### Trigger Mapping
- Frontend deploy trigger: description
- Backend deploy trigger: description
- Migration trigger: Automatic on deploy / Manual / Not detected
- Are frontend + backend deployed together: Yes / No — description

### Staging → Production Parity
- Pipelines symmetrical: Yes / No — describe differences
- Idempotent promotion: Yes / No — describe manual steps if any
- Environment-specific divergence: list any steps that differ

### Security in CI
- Secrets source: GitHub Secrets / hardcoded — list any hardcoded values found
- Third-party actions pinned to SHA: Yes / No / Partial — list unpinned
- Runner permissions: least-privilege / default / not configured
- Image scanning in CI: tool / None detected

### Migration Automation
- Migrations in CI: Automatic / Semi-automatic / Manual / Not detected
- Rollback on migration failure: Yes / No

### Notifications
- Failure notification: channel / None detected
- Production deploy notification: channel / None detected

### Gaps Detected
- list factual gaps (no grades here — grades go in arc-scoring.md)
```
