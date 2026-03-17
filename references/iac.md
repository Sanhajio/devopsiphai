# IaC Domain Audit

Read this file when auditing infrastructure as code. Feeds into the **Control** ARC pillar.

---

## What to investigate

### IaC Coverage
- What is managed by IaC vs manually provisioned?
  - List each infrastructure component and its provisioning method
- What IaC tool is in use?
  - Terraform, Pulumi, CloudFormation, CDK, Ansible, Helm, Nix, none

### Networking Schema
- Map the network topology from IaC or infer from config/CI:
  - VPC / VNet structure
  - Subnets (public vs private)
  - Load balancers / ingress
  - Security groups / firewall rules
  - CDN configuration
  - DNS configuration
- If no IaC exists, infer from: docker-compose networks, CI deploy targets,
  environment variables referencing endpoints, README architecture diagrams

### Region Mapping
- List all cloud regions in use per provider:
  - AWS: list regions per service
  - GCP: list regions per service
  - Azure: list regions per service
- Note cross-region dependencies (e.g. app in eu-west-3, S3 in eu-north-1)
- Note cross-cloud dependencies (e.g. AWS compute + GCP vector DB)

### Self-Hosted vs SaaS Analysis
For each service/dependency, classify:
- **Can be self-hosted**: open-source alternative exists, feasible to run
- **Typically self-hosted**: commonly run on own infra
- **Requires SaaS**: no viable self-hosted alternative, or migration cost too high

Examples:
- Supabase → can be self-hosted (Docker Compose available)
- Clerk → hard to replace (proprietary), but Keycloak is an alternative
- Pinecone → can be replaced with Qdrant, Weaviate, pgvector
- Vercel → can be replaced with Coolify, Dokku, self-hosted Next.js
- OpenAI/Anthropic → cannot be fully self-hosted (model weights not public)
- Sentry → can be self-hosted (open-source)
- GitHub Actions → can be replaced with Gitea + Forgejo Actions, self-hosted runners

### IaC Quality (if IaC exists)
**Terraform:**
- Remote state configured with locking?
- Provider versions pinned in `required_providers`?
- Modules versioned (not pointing to `main`)?
- `terraform fmt` and `terraform validate` in CI?

**Helm:**
- `values.yaml` defaults safe for production?
- Chart dependencies pinned?
- `helm diff` used before upgrades?
- Resource requests/limits defined?
- Separate values files per environment?

**Nix:**
- `flake.lock` committed?
- `devShell` for reproducible local dev?
- NixOS modules for service config?
- `nixos-rebuild dry-activate` in CI?

---

## Output Format

```
## IaC Audit

### IaC Coverage
| Component | Provisioning method |
|---|---|
| component | IaC (tool) / Manual / Unknown |
_(list all infrastructure components)_

### Networking Schema
- **Topology**: description of inferred or documented network layout
- **VPC/Subnets**: described / Not detectable
- **Load balancer / Ingress**: name / Not detected
- **Security groups / Firewall**: described / Not detectable
- **CDN**: provider / Not detected
- **DNS**: provider / Not detected
- **Diagram**: _ASCII or description of connectivity_
  ```
  [Frontend: Vercel] → [Backend: AWS App Runner eu-west-3]
                              ↓
                    [Supabase Postgres: cloud]
                    [Pinecone: GCP us-east1]
                    [S3: eu-north-1]
  ```

### Region Mapping
- AWS: service → region
- GCP: service → region
- Cross-region dependencies: list
- Cross-cloud dependencies: list

### Self-Hosted vs SaaS Analysis
| Service | Current | Self-hostable | Alternative |
|---|---|---|---|
| name | SaaS/self-hosted | Yes/No/Partial | alternative or N/A |

### IaC Quality (if applicable)
- Tool: name
- State management: configured / not configured
- Versions pinned: Yes / No
- CI validation: Yes / No
- Environment separation: Yes / No

### Gaps Detected
- list factual gaps
```
