# Recurring maintenance runbook

> Created 2026-07-14 (tech-artifact review). **Purpose:** one place the operator opens on a schedule to keep both MicLine environments current and healthy — dependencies, toolchain, Cloudflare products, security posture, cost signals. It **wraps** (never duplicates) build-guide [02 §2.9](../build-guide/02-setup.md) — that section owns the toolchain-update *how*; this file owns the *when* and the operational checks around it. Rows are tagged with when they become applicable; skip what doesn't exist yet. Anomalies found here escalate via `docs/runbooks/security-incident-response.md`.

## Weekly (~10 min) — from F000

- [ ] **Dependabot PRs** (grouped weekly): review + merge through the chore lane (build-guide 05 §7.12). A major-version bump gets its release notes read, not rubber-stamped.
- [ ] **GitHub → Security tab:** CodeQL alerts, secret-scanning alerts, Dependabot alerts — triage to zero (fix, dismiss-with-reason, or issue).
- [ ] **Actions:** last `ci.yml` / `deploy-dev.yml` runs green; Deployments sidebar shows DEV/PRD where you expect them.
- [ ] **Health check (from M1/F008):** `curl -s https://dev.micline.app/health` and `https://micline.app/health` → 200 (AF-8; a 503 names the failing service — escalate via the incident runbook).
- [ ] **Cloudflare dashboard glance** (from first deploy): Workers requests, DO duration, email sends vs. the 3,000/mo allotment, D1/DO storage growth — anything that doesn't match `docs/engineering/runcost-estimate.md` expectations is investigated (a DO that never hibernates is the classic silent cost driver).

## Monthly (~20 min) — from F000

- [ ] Run the **[02 §2.9](../build-guide/02-setup.md) machine-tools block** (brew tools, Claude Code, `specify self check`).
- [ ] `pnpm outdated` at the repo root: note pending **repo-pinned** bumps (Node `.node-version`, pnpm `packageManager`, wrangler, `@cloudflare/vite-plugin`) and schedule deliberate chore-lane PRs. **Miniflare note:** miniflare is maintained inside the `cloudflare/workers-sdk` monorepo (releases there); the standalone `cloudflare/miniflare` repo is archived — don't watch the wrong repo.
- [ ] Skim the **Cloudflare changelog** for the products in use (workers, d1, durable-objects, kv, email-service, turnstile, + access from M6, r2 from M7): <https://developers.cloudflare.com/changelog/> — pricing or beta-status changes feed `runcost-estimate.md` and tech-context §19/§22.
- [ ] Email Service standing: sending quota level (auto-scaling), bounce/complaint signals in the dashboard.

## Per milestone kickoff — always (this is the [07 §9](../build-guide/07-m5-to-m13.md) ritual's ops half)

- [ ] Full 02 §2.9 ritual incl. Spec Kit upgrade path.
- [ ] Guide-freshness sweep ([10 §D](../build-guide/10-whats-next.md)): re-verify the build guide's point-in-time facts; update the [09](../build-guide/09-troubleshooting.md) assumptions log.
- [ ] Work the milestone's due rows in **tech-context §22** (verify-at-implementation list).
- [ ] Re-verify `runcost-estimate.md` prices; from M12 on, reconcile against the OPS-02 measured model instead.

## Quarterly (~15 min) — from first deploy

- [ ] **Account security review:** GitHub + Cloudflare + operator-mailbox sessions/tokens/2FA devices — anything unfamiliar → incident runbook §1.3.
- [ ] **`micline-ci` API token:** confirm scopes still minimal (Workers/D1/KV edit); rotate if it's grown old enough to make you uncomfortable (procedure: incident runbook §2.3 — cheap, safe).
- [ ] **Resource inventory:** every `micline*`-prefixed resource still tagged `project:micline` + `env:*` (tech-context §3; conventions in `resource-bootstrap.md`); nothing orphaned from experiments.
- [ ] Access policy allowlist review (from M6, Access wall mode — AF-7; in password mode instead: confirm `ADMIN_PASSWORD` is set and the Access nudge still shows); Turnstile widget hostnames still exactly `micline.app` + `dev.micline.app` (from M4).
- [ ] Interaction limits renewal if pre-launch (02 §2.4 step 5, max 6 months); re-check at/after the M4 gate.

## How to run this file

Top to bottom for the due cadence; tick mentally (this file stays a template — no state lives here, per the PROGRESS.md-only rule). Log anything done by hand in a web UI as one line in `manual-config-log.md`. If a check fires, don't fix ad hoc: chore-lane PR, bugfix lane, or the incident runbook — the lanes in build-guide [05 §7.12](../build-guide/05-feature-loop.md) decide.
