# Phase 2 — Domain Audit

This phase performs deep, domain-specific audits based on what was discovered in
`preliminary-audit.md`. Each domain is audited by reading its reference file.

## Rules

- Load each reference file only when actively auditing that domain
- Skip domains where no relevant files were found in Phase 1
- State findings factually — scoring and grades are reserved for `arc-scoring.md`
- One subagent per domain, run in parallel

## Domains

For each domain detected in Phase 1, read the corresponding reference file and run
the audit. Report findings per domain before moving to `arc-scoring.md`.

| Domain | Reference file | Trigger condition |
|---|---|---|
| CI/CD | `references/cicd.md` | Any `.github/workflows/`, `.gitlab-ci.yml`, `Jenkinsfile`, `azure-pipelines.yml` |
| Containers | `references/containers.md` | Any `Dockerfile*`, `docker-compose*.yml` |
| IaC | `references/iac.md` | Any `*.tf`, `Chart.yaml`, `*.nix`, `flake.nix`, `helmfile.yaml`, `*.yaml` with k8s kind |
| Observability | `references/observability.md` | Any Prometheus rules, Grafana JSON, OTel config, Loki config |
| Security | `references/security.md` | Always run — baseline security audit on every project |
| Onboarding | `references/onboarding.md` | Always run — scored against every project |

## Output Format per Domain

```
## Domain: [Name]

### Findings
- [finding description — factual, no grade yet]

### Evidence
- File: `path/to/file` — what was found
```
