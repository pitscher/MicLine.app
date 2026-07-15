# F008 — Platform config, flags & hardening baseline

**Milestone:** M1 — Deployable foundation · **Target version:** `v0.1.0` · **Implements:** FND-03, FND-04, FND-06, FND-07
**Depends on:** F000. **Gate-2 note:** confirmed as a separate package (not folded into F000).

## User value

The operator can adjust every platform value and pull the two emergency brakes (event-creation kill-switch, maintenance banner) **without a code change or redeploy**, from `v0.1.0` on — and the security/privacy schema that M6–M8 reach back into (role, audit, soft-delete) exists before the first product row is ever written, so no later milestone needs a painful retrofit.

## In scope

- **Central platform configuration (FND-04):** the KV `CONFIG` mechanism per tech-context §13 — one zod-validated document with committed defaults covering every value in the product brief's "Limits & platform values" table (create-ahead window, caps, question-length bounds, announcement lifetime, reconnect grace, purge offsets, text caps, export-link validity, curated-limit default); malformed config falls back to defaults, never crashes. Every key ships with an operator-facing description (zod `.describe()`), rendered wherever config is shown (runbook + M6 admin UI) — the operator must always know exactly what a value does.
- **Kill-switch flags:** `flag:event-creation-disabled` + `flag:maintenance-banner` as KV keys; operator runbook `docs/runbooks/config-operations.md` — one entry per key: exact wrangler CLI command, committed default, description (M1 operator surface); the M6 admin UI and M8 invite-only flag later write the same keys through the same service.
- **Global log-level control (FND-06):** `platform:log-level` (debug ↔ critical), operator-adjustable, ≤ ~90 s propagation (accepted), consumed by the structured logger everywhere (Worker + DO).
- **Hardening baseline (FND-03):** `users` stub table (no role column — AF-7: the admin is not a product user), append-only **audit-log table** (privileged admin/entitlement actions ONLY — REJ-09 is constitutionally narrow), soft-delete semantics, structured JSON logging with request correlation, WS-origin-allowlist middleware seam (enforced when sockets land in F003), keyless **`ops_counters`** table + stats-service seam (AF-3 — first writers arrive in M2).
- **Health endpoint (FND-07 — AF-8):** `GET /health` gated by the deploy-time var `HEALTH_ENDPOINT_ENABLED` (a Wrangler var, not KV config — must work when KV is the failure); 200 only if D1 + KV respond, else 503; minimal body, `no-store`, never behind auth, per-IP burst ceiling generous for 1/min monitors; reports **infrastructure** health (tech-context §16).

## Out of scope

Admin UI wrapper for flags (M6 / ADM-07) · invite-only flag activation (M8 / ENT-05) · the M7 emergency-stop flag (explicitly a different, stronger mechanism — tech-context §10/§13) · any consumer of the audit log (first writers arrive M6/M8).

## Dependencies & seams

Every value this package homes is consumed later (F001 reads create-ahead/reschedule/length bounds; F003 reads announcement lifetime + grace; F010 reads purge offsets). The audit-log and soft-delete shapes are M6/M8 demands pulled into M1 by design.
