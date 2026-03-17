# Observability Domain Audit

Read this file when auditing observability. Feeds into the **Reporting** ARC pillar.

---

## What to investigate

### Metrics
- Are Prometheus metrics exported? (`/metrics` endpoint, custom counters/gauges)
- Are SLOs/SLAs defined as recording rules (not just dashboard expressions)?
- Are alerting rules defined in codebase?
  - Prometheus rules files (`*.rules`, `*.yaml` with `groups:` and `alert:`)
  - Grafana alert JSON
  - PagerDuty config
  - Not in codebase — UI only
- Do alerts have runbook links or descriptions?

### Dashboards
- Are dashboards version-controlled?
  - Grafana JSON files in repo
  - Grafonnet / grafanalib / Grafana Operator
  - Datadog dashboard JSON
  - None — managed in UI only
- Are dashboards organized by service/team?
- Is there a golden signals dashboard per service?
  (latency, traffic, errors, saturation)

### Logs
- Is structured logging in use? (JSON format, consistent levels)
- Is there a log aggregation backend? (Loki, Elastic, Datadog, CloudWatch)
- Are log levels consistent and queryable?
- Is log retention policy defined?
- Are critical error patterns alertable via log-based alerts?

### Tracing
- Is distributed tracing instrumented?
- Are trace IDs propagated across service boundaries?
- Are trace IDs correlated with log output?
- Is there an OTel Collector configured?
  - Where: path in repo
  - Exporters: list

### User Action Audit Trail
- Is user action logging present?
- What system captures it?
  - Core application database (which table/schema)
  - Supabase audit log (pg_audit extension or built-in)
  - Third-party: Mixpanel, Segment, Amplitude, Datadog
  - None detected
- What events are logged?
  - Auth events (login, logout, token refresh)
  - Data mutations (create, update, delete)
  - Admin actions
  - API calls
- Is the audit trail queryable and retained?

### Error Tracking
- Is error tracking configured? (Sentry, Datadog, Rollbar, Bugsnag)
- Are errors enriched with context (user ID, request ID, environment)?
- Is PII capture enabled or disabled?
- Are error alerts configured?

### Monitoring Coverage
- Which services have observability? Which do not?
- Are health check endpoints present? (`/health`, `/ready`, `/live`)
- Are external dependencies monitored? (DB, third-party APIs)

---

## Output Format

```
## Observability Audit

### Metrics
- Prometheus endpoint: path / None detected
- Recording rules in codebase: Found at path / None
- Alerting rules in codebase: Found at path / None — UI only
- Alert runbooks: Present / Absent

### Dashboards
- Dashboards in repo: Found at path (tool) / None — UI managed
- Dashboard organization: By service / Ad hoc / Not applicable
- Golden signals coverage: Present / Partial / Absent

### Logs
- Backend log format: JSON / Plaintext / Mixed
- Frontend log format: JSON / Plaintext / Mixed
- Log aggregation: backend / None
- Log retention policy: Defined / Not defined
- Log-based alerts: Present / Absent

### Tracing
- Instrumented services: list
- Cross-service trace propagation: Yes / No / Partial
- Trace ↔ log correlation: Yes / No
- OTel Collector: Found at path / Not detected
  - Exporters: list

### User Action Audit Trail
- Present: Yes / No
- System: name (core DB / Supabase / third-party)
- Events logged: list or "Not detectable"
- Queryable + retained: Yes / No / Unknown

### Error Tracking
- Tool: name / None
- Context enrichment: Yes / Partial / No
- PII capture: Enabled / Disabled / Unknown
- Error alerts: Configured / Not configured

### Coverage Gaps
- Services without observability: list
- Health endpoints: Present / Absent per service

### Gaps Detected
- list factual gaps
```
