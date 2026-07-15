# Cloudflare resource bootstrap

> **Status: convention + command stub, committed 2026-07-14 (tech-artifact review). F000 completes it** — replaces the placeholders below with the exact commands actually run and documents where every returned id landed in `wrangler.jsonc`. Purpose (build-guide [02 §2.8](../build-guide/02-setup.md), layer 2): the repo alone must be able to recreate every Cloudflare resource; this file is that recipe. Command syntax is re-verified at F000 against current wrangler docs (tech-context §22).

## Naming & tagging conventions (binding — tech-context §3)

The Cloudflare account is shared with other projects; every MicLine resource must be distinguishable for tracking and billing attribution:

| Resource | DEV | PRD | Created |
|---|---|---|---|
| Worker | `micline-dev` | `micline` | F000 (first deploy per env) |
| D1 database | `micline-dev` | `micline` | F000 (below) |
| KV namespace (`CONFIG` binding) | `micline-config-dev` | `micline-config` | F000 (below) |
| CI API token | `micline-ci` (one token, both envs) | — | 02 §2.7 row 1 |
| Turnstile widget | `micline` (one widget, both hostnames) | — | 02 §2.7 row 3 (M4) |
| Access application | `micline-admin` (both hostnames' `/admin*`) | — | row 5 (M6) |
| R2 bucket | `micline-exports-dev` | `micline-exports` | M7 only — do not create earlier |

**Resource Tags** (Cloudflare Resource Tagging, public beta): every taggable resource (Workers, D1, KV, R2, zone-level where sensible) gets `project:micline` + `env:dev` or `env:production`, applied right after creation — enables dashboard filtering and usage/billing attribution. API-first (a PUT replaces the whole tag set — always write the full set); dashboard: Manage Account → Resource Tagging. Each tagging action is logged in `manual-config-log.md` like any manual step.

## D1 databases *(placeholders — F000 fills real output)*

```bash
pnpm wrangler d1 create micline-dev
pnpm wrangler d1 create micline
```

→ paste each returned `database_id` into `wrangler.jsonc` (`env.dev` / `env.production` → `d1_databases`). Record the actual ids here: `⟨F000: dev id⟩` · `⟨F000: prd id⟩`.

## KV namespaces *(placeholders — F000 fills real output)*

```bash
pnpm wrangler kv namespace create micline-config-dev
pnpm wrangler kv namespace create micline-config
```

→ paste each returned namespace id into `wrangler.jsonc` (`CONFIG` binding per env). Record: `⟨F000: dev id⟩` · `⟨F000: prd id⟩`.

## Tagging *(F000 records the exact API calls used)*

```text
⟨F000: Resource Tagging API calls / dashboard steps applying project:micline + env:* to the workers, D1 databases, and KV namespaces above⟩
```

## R2 buckets (M7 — placeholder, deliberately empty until then)

```text
⟨M7: pnpm wrangler r2 bucket create micline-exports(-dev) + lifecycle rule for export auto-delete + tags⟩
```

## What this file never contains

Secrets, tokens, or ids of *other* projects. Secrets are never IaC (constitution Art. I) — the rebuild path for those is the secrets matrix (tech-context §3) + `openssl rand -base64 32`. The residual console-only clicks live in [02 §2.7](../build-guide/02-setup.md) and are logged in `manual-config-log.md`; M12's rebuild-from-zero runbook (OPS-05) compiles all of it.
