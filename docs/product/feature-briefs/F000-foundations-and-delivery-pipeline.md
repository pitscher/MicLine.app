# F000 — Foundations & delivery pipeline

**Milestone:** M1 — Deployable foundation · **Target version:** `v0.1.0` · **Implements:** FND-01, FND-02
**Depends on:** nothing — deliberately first and alone. **Ceremony:** reduced (skip clarify-depth and checklist), but the plan MUST read `docs/build-guide/03-github-operations.md` as binding design input and `docs/engineering/tech-context.md` §1–§5, §17.

## User value

Every later package lands on rails: from the first line of product code, a merge to `main` is on DEV minutes later, a published Release is on PRD after one deliberate approval, secrets cannot enter the public tree, and an unextracted UI string fails CI. The repo alone (plus five logged console rows) can recreate the whole infrastructure — the operator's rebuild-from-zero requirement starts being true here.

## In scope

- **Monorepo & app skeleton:** pnpm/turbo workspace (`apps/web`, `packages/shared`, `packages/ui`); `.node-version` + `packageManager` pins; React Router framework mode (major decided here against the current official Cloudflare template — recorded in tech-context §1) + `@cloudflare/vite-plugin` + Tailwind v4; Hono mounted for `/api`+`/ws`; strict tsconfig; Biome; Vitest + workers-pool smoke test; security-header baseline middleware (CSP report-only; `Referrer-Policy: no-referrer` and **default-deny indexing** — `X-Robots-Tag: noindex` on every response, both environments; F007 later allowlists the marketing surfaces on PRD — AF-1/AF-6) + request-body size caps on API routes (AF-1).
- **i18n plumbing (FND-02):** Lingui with `en`+`de`, extraction + catalog compile wired into `build` so unextracted strings fail CI; every user-facing string through macros from the first component.
- **Environments:** `wrangler.jsonc` per tech-context §3 (names/ids only, never secrets); `.dev.vars.example`; **complete** the committed `docs/runbooks/resource-bootstrap.md` stub (real D1/KV creation commands + ids per env) and apply its naming (`micline` prefix) + Resource-Tagging (`project:micline`, `env:*`) conventions to everything created (tech-context §3).
- **GitHub plumbing:** `ci.yml` (five named checks incl. gitleaks), `deploy-dev.yml`, `deploy-prd.yml` (tag checkout, `production` approval), `.github/release.yml`, issue/PR templates (incl. AI-disclosure checkbox), `SECURITY.md`, workflow hardening (SHA pins, least-privilege permissions), committed pre-commit gitleaks hook, Dependabot (npm+actions weekly grouped), CodeQL, README badges + links.
- **Agent plumbing:** nested `CLAUDE.md` stubs in `apps/web/worker/` and `packages/shared/`.

## Out of scope

Platform config/flags/log-level and all hardening schema (F008) · any product feature or page beyond a hello route · marketing README content (F007 era).

## Exit checklist (definition of done)

(a) `pnpm dev` serves a hello page through the Worker;

(b) CI green on the F000 PR;

(c) planted fake secret blocked by the hook;

(d) manual row 1 done (Workers Paid, CI token, GitHub environments + secrets) → `deploy-dev.yml` re-run → hello page live on `dev.micline.app`;

(e) branch protection ON (five required checks, PRs, linear history).
