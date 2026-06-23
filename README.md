# TenantForge

> **Provision tenants. Not config drift.** Production-grade Terraform modules for
> Microsoft Entra ID — OIDC federation, multi-tenant apps, and environment
> matrices, with zero hardcoded secrets.

[![License: MIT](https://img.shields.io/badge/License-MIT-success.svg)](LICENSE)
[![Part of TenantFleet](https://img.shields.io/badge/TenantFleet-Mind-2e7d32)](https://t-granlund.github.io/tenantfleet/)
[![Live demo](https://img.shields.io/badge/demo-live-1f6feb)](https://t-granlund.github.io/tenantforge/)
[![Terraform](https://img.shields.io/badge/IaC-Terraform-7b42bc)](#architecture)

TenantForge is the infrastructure-as-code **Mind** pillar of the
[TenantFleet](https://t-granlund.github.io/tenantfleet/) ecosystem. It is a set
of composable Terraform modules that turn "stand up another tenant" from a
copy-paste-and-pray exercise into a single, parameterized module call.

---

## Why it exists

Every new brand, client, or portfolio company arrives with its own Entra ID
estate, and teams end up rewriting the same Terraform for each one. TenantForge
abstracts those patterns — app registrations, RBAC, federated credentials,
environment matrices — so you **provision, not puzzle**.

## What it delivers

| Capability | What it does |
| --- | --- |
| **Modular Terraform** | Composable modules for app registrations, RBAC assignments, and federated credentials. |
| **OIDC federation** | GitHub Actions → Azure via workload identity federation. No client secrets stored in CI. |
| **Multi-tenant app** | Single app-registration model that scales across MSP, PE portfolio, and franchise tenants. |
| **Environment matrix** | GitHub Environments per tenant with matrix deployment strategies and approval gates. |
| **State isolation** | Per-tenant backend storage with naming conventions that prevent cross-tenant leakage. |
| **Cost tagging** | Automatic Azure cost tags per brand, portfolio, and environment for chargeback visibility. |

**At a glance:** 6 Terraform modules · ∞ OIDC credentials · 0 hardcoded
secrets · &lt; 10m first deploy.

## Install

```bash
git clone https://github.com/t-granlund/tenantforge.git
cd tenantforge
terraform init
```

You will need the [Terraform CLI](https://developer.hashicorp.com/terraform) and
an Azure subscription. Authentication is via **OIDC workload identity
federation** — configure your GitHub repo/environment as a federated credential
rather than storing a client secret.

## Usage

Compose a tenant from the modules. A minimal app registration:

```hcl
module "tenant_core" {
  source    = "github.com/t-granlund/tenantforge//modules/entra-app-registration"
  tenant_id = var.tenant_id
  app_name  = "dashboard-${var.brand}"

  redirect_uris = ["https://${var.brand}.app/auth/callback"]

  # Zero secrets — OIDC federation only
  enable_oidc_federation = true
  github_org             = var.github_org
  repo_name              = var.repo_name
}
```

One module. Any tenant. Add a brand by adding a variable set — not by forking
the infrastructure.

## Architecture

```
GitHub Actions ──(OIDC federation, no secrets)──► Azure / Entra ID
      │
      ▼
 Terraform modules
   ├── entra-app-registration   (multi-tenant app + redirect URIs)
   ├── rbac-assignments         (role assignments per persona)
   ├── federated-credentials    (workload identity per repo/env)
   ├── environment-matrix       (GitHub Environments + approval gates)
   ├── state-backend            (per-tenant isolated state)
   └── cost-tagging             (brand/portfolio/env tags)
```

- **Per-tenant state isolation** keeps one brand's state from ever touching
  another's.
- **Environment matrices** drive repeatable, gated deploys across dev → prod.
- **Cost tagging** makes chargeback and showback fall out of the apply.

## Security

- **Zero hardcoded secrets** — workload identity federation end to end.
- **Least privilege** — scoped role assignments per module.
- **Isolation by construction** — naming conventions + separate state prevent
  cross-tenant leakage.

## Part of the TenantFleet ecosystem

| Repo | Pillar | Focus |
| --- | --- | --- |
| [TenantFleet](https://t-granlund.github.io/tenantfleet/) | — | Root governance framework |
| [HubForge](https://t-granlund.github.io/hubforge/) | Mind | Azure SWA + Entra deployment templates |
| [EntraGroups](https://t-granlund.github.io/entragroups/) | Body | Group lifecycle + persona RBAC |
| **TenantForge** | Mind | Terraform tenant provisioning |
| [DNSGuard](https://t-granlund.github.io/dnsguard/) | Spirit | Domain + DMARC intelligence |
| [RampGuard](https://t-granlund.github.io/rampguard/) | Mind | Finance compliance + spend |
| [SharePointAgent](https://t-granlund.github.io/sharepoint-agent/) | Body | SharePoint indexing + Teams |

See the full ecosystem on the portfolio:
[tylergranlund.com/work#ecosystem](https://tylergranlund.com/work#ecosystem).

## License

MIT — see [LICENSE](LICENSE). Fork it, deploy it, make it yours.
