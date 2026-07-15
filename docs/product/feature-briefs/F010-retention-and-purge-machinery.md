# F010 — Retention & purge machinery

**Milestone:** M4 — Public-beta readiness · **Target version:** `v0.4.0` · **Implements:** EVT-06, EVT-07, OPS-03
**Depends on:** F003 (the DO that self-purges), F001 (lifecycle fields).

## User value

The product's boldest privacy promise — *event content dies 3 days after the end* — becomes enforced, tested truth **before** the first stranger relies on it (P3-6). Aggregates survive only in a form that cannot point back at anyone, so the operator's stats never become a liability.

## In scope

- **3-day content purge (EVT-07):** DO self-purge at end+3 d via its multiplexed alarm (real deletion — DO storage has no recovery tier); in-product countdown surfaces (my-events; recap consumes it in F011); **no reminder email** (T7-4).
- **Event stats aggregates (EVT-06):** snapshot at event end into the keyless aggregates store — unlinkable by construction; survives account/event deletion (D4).
- **Retention enforcement for every lifetime (OPS-03):** cron sweeper per tech-context §10 — auth tokens, inactive accounts (1 month; carve-outs: future event scheduled, M8 wallet), audit log (platform config; default 12 months), KV code cleanup, defensive DO re-arm.
- **The rule:** every row of the data-lifecycle table maps to exactly one enforcement job **and one test proving deletion actually happens**.
- **M7 seams only:** purge sites check the emergency flag (pause + 24 h resume grace machinery itself is M7).

## Out of scope

Emergency stop behavior (M7 / ADM-08) · export (M5 / STD-05) · verification/monitoring audit of enforcement (M12 / OPS-04) · account self-service UI (F011 — the inactivity *job* lives here, its UX there).

## Dependencies & seams

Purge offsets come from platform config (F008). The aggregates write is the model for D4's aggregation-at-deletion pattern reused by F011's account deletion.
