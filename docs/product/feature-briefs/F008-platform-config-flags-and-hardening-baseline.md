# F008 — Platform config, flags & hardening baseline

**Milestone:** M1 — Deployable foundation · **Target version:** `v0.1.0` · **Implements:** FND-03, FND-04, FND-06
**Depends on:** F000. **Gate-2 note:** confirmed as a separate package (not folded into F000).

## User value

The operator can adjust every platform value and pull the two emergency brakes (event-creation kill-switch, maintenance banner) **without a code change or redeploy**, from `v0.1.0` on — and the security/privacy schema that M6–M8 reach back into (role, audit, soft-delete) exists before the first product row is ever written, so no later milestone needs a painful retrofit.

## In scope

- **Central platform configuration (FND-04):** the KV `CONFIG` mechanism per tech-context §13 — one zod-validated document with committed defaults covering every value in the product brief's "Limits & platform values" table (create-ahead window, caps, question-length bounds, announcement lifetime, reconnect grace, purge offsets, text caps, export-link validity, curated-limit default); malformed config falls back to defaults, never crashes.
- **Kill-switch flags:** `flag:event-creation-disabled` + `flag:maintenance-banner` as KV keys; operator runbook with exact wrangler CLI commands (M1 operator surface); the M6 admin UI and M8 invite-only flag later write the same keys through the same service.
- **Global log-level control (FND-06):** `platform:log-level` (debug ↔ critical), operator-adjustable, ≤ ~90 s propagation (accepted), consumed by the structured logger everywhere (Worker + DO).
- **Hardening baseline (FND-03):** `users.role` column + stub table, append-only **audit-log table** (privileged admin/entitlement actions ONLY — REJ-09 is constitutionally narrow), soft-delete semantics, structured JSON logging with request correlation, WS-origin-allowlist middleware seam (enforced when sockets land in F003).

## Out of scope

Admin UI wrapper for flags (M6 / ADM-07) · invite-only flag activation (M8 / ENT-05) · the M7 emergency-stop flag (explicitly a different, stronger mechanism — tech-context §10/§13) · any consumer of the audit log (first writers arrive M6/M8).

## Dependencies & seams

Every value this package homes is consumed later (F001 reads create-ahead/reschedule/length bounds; F003 reads announcement lifetime + grace; F010 reads purge offsets). The audit-log and soft-delete shapes are M6/M8 demands pulled into M1 by design.
