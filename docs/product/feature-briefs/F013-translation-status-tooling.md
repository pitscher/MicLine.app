# F013 — Translation status tooling (Lunaria)

**Milestone:** M4 — Public-beta readiness · **Target version:** `v0.4.0` · **Implements:** FND-05
**Depends on:** F000 (Lingui catalogs exist). **Non-gating** — may slot in any time after the M2 work, ideally before UI strings multiply in M3/M4. Light ceremony.

## User value

The operator (and future contributors) can see per locale which catalogs are done, outdated, or missing — on every PR via an Action comment, optionally on a static dashboard — without any external service, account, or standing cost. Complements what already exists for free: F000's build fails CI on unextracted strings; `pnpm lingui extract` prints missing counts on demand.

## In scope

- The Lunaria loop (EmDash pattern): git-history-based status tracking, the official GitHub Action commenting translation impact on PRs, optionally `lunaria build`'s static dashboard (GitHub Pages is acceptable for tooling, or skip hosting and let PR comments be the dashboard).
- The tracking-mode decision against our Lingui catalog format: `universal` file-level tracking of `.po` catalogs (simplest) vs switching Lingui to JSON catalogs for per-key missing lists — decided in this loop and recorded.

## Out of scope

The rented TMS (M10 / STD-11 — which replaces-or-feeds this, decided at the M10 kickoff) · making the Action a required check (**informational only, forever**) · any translation content work.

## Dependencies & seams

Lunaria is beta — fetch its docs fresh in the loop (getting-started, configuration, github-action). If a dashboard is hosted, it deploys from CI, never by hand.
