# Security incident response runbook

> Created 2026-07-14 (tech-artifact review). **Purpose:** when the operator *suspects* a security incident — leaked credential, odd traffic, unexpected email volume, a login that wasn't you, a report via private vulnerability reporting — this runbook is executed top to bottom, **before** (and possibly instead of) reaching for the admin UI's emergency controls. Sections are tagged with the milestone from which they apply; run what exists. Companion documents: pipeline threat map (build-guide 03 §G) · accepted residual risks (tech-context §15) · GDPR breach procedure (M12 OPS-06 compliance pack — the legal side; this file is the operational side).
>
> **Rules while responding:** never print or paste a secret anywhere (public repo, CI logs, issues); log every action taken as one line in `docs/runbooks/manual-config-log.md` with a timestamp; prefer over-rotation to under-rotation — every rotation below is designed to be safe to run "just in case".

## 0 · Standing prevention posture (verify quarterly — see `docs/runbooks/maintenance.md`)

- 2FA (hardware key/passkey) on **GitHub**, **Cloudflare**, *and the operator mailbox* — the mailbox is the root of trust: it anchors your magic-link logins, `ADMIN_EMAILS`, and the M6 Access one-time PIN.
- Secrets exist only in `.dev.vars` (local), Wrangler secrets, and GitHub Environment secrets. If you ever see a secret-shaped string anywhere else, treat it as leaked and jump to §2.
- GitHub secret scanning + push protection + private vulnerability reporting stay ON; Actions fork-PR approval stays strictest.

## 1 · Assess (≤15 minutes — classify, don't investigate deeply yet)

1. What is the signal? Workers Logs (odd request patterns, auth spikes) · Cloudflare dashboard usage graphs (request/DO/email anomalies) · Email Service send counts vs. expectation · GitHub security tab (secret-scanning or CodeQL alert, unfamiliar workflow run) · account notification emails · a vulnerability report.
2. Classify into one (or more) of: **A leaked credential/secret** · **B abusive traffic/cost attack** · **C suspected account takeover (GitHub/Cloudflare/mailbox)** · **D suspected data breach (someone read or exfiltrated stored data)**.
3. If **C**: change that account's password + rotate its 2FA/sessions FIRST (GitHub → Settings → Sessions; Cloudflare → My Profile → Active sessions; mailbox provider equivalent), then continue below — everything else is downstream of the two accounts.

## 2 · Contain — rotate credentials (from M1/first deploy; safe to run on suspicion)

Rotations are ordered by blast radius (smallest user impact first). All commands run from the repo root.

**2.1 `SESSION_SECRET` (from M2 · F002)** — invalidates every moderator session (global logout; users just log in again via magic link; participant cookies re-mint on next visit). *This is the designed lever — there is no per-session revocation by design (tech-context §11.2).*

```bash
openssl rand -base64 32 | pnpm wrangler secret put SESSION_SECRET --env dev
openssl rand -base64 32 | pnpm wrangler secret put SESSION_SECRET --env production
# takes effect within seconds; also update .dev.vars locally if it was the leaked copy
```

**2.2 `TURNSTILE_SECRET_KEY` (from M4 · F006)** — dashboard → Turnstile → widget `micline` → rotate secret key, then:

```bash
pnpm wrangler secret put TURNSTILE_SECRET_KEY --env dev        # paste new value at the prompt
pnpm wrangler secret put TURNSTILE_SECRET_KEY --env production
```

**2.3 `CLOUDFLARE_API_TOKEN` (from M1 · F000)** — dashboard → My Profile → API Tokens → `micline-ci` → **Roll** (or delete + recreate from the same template: Edit Cloudflare Workers + D1:Edit + Workers KV Storage:Edit). Update the secret in **both** GitHub Environments (`dev` AND `production`): repo → Settings → Environments. CI deploys fail until both are updated — that's the confirmation the old token is dead.

**2.4 Other Wrangler secrets** (`FEEDBACK_EMAIL` from M4, `ADMIN_EMAILS` from M6) — same `pnpm wrangler secret put ⟨NAME⟩ --env ⟨e⟩` pattern; rotate if the incident touched them.

**2.5 Access sessions (from M6)** — Zero Trust dashboard → revoke active Access sessions for the `micline-admin` app; review the allowlist policy for entries you didn't add.

**2.6 If a secret hit the public git tree:** GitHub push protection/secret scanning should have blocked it — if one slipped through anyway, **rotation first, scrubbing second** (constitution Art. I: a leak is an incident, not a cleanup chore). After rotating, rewrite history only if the value would otherwise stay harvestable, and run `gitleaks git` over the full history to confirm nothing else lurks.

## 3 · Contain — throttle the platform (choose the smallest sufficient lever)

| Lever | From | How |
|---|---|---|
| Event-creation kill-switch + maintenance banner | M1 (F008) | exact commands per key: `docs/runbooks/config-operations.md` (F008 creates it); mechanism: tech-context §13 |
| Tighten rate-limit config values | M4 (F006) | same config surface |
| WAF rule (block/challenge a pattern or ASN) · Turnstile mode escalation | any time | Cloudflare dashboard (zone → Security); documented fully in the M12 OPS-05 runbook — until then: smallest scope, log the rule, remove it when done |
| Disable GitHub Actions entirely (suspected pipeline compromise) | any time | repo → Settings → Actions → disable; review recent workflow runs + Deployments sidebar before re-enabling |
| Force-end / ban / soft-delete a specific event or moderator | M6 (admin UI) | ADM-03 tooling |
| **Emergency stop** (total platform freeze, reversible) | M7 (admin UI) | ADM-08 — in the trigger modal, **skip the auto-export if you suspect data compromise** (its own design rule) |
| **Delete all data** (irreversible GDPR/breach nuke) | M7 (admin UI) | ADM-09 — last resort, operationally preceded by the emergency stop |

Note the FR-1 window: before M6 there is **no** in-product lever that ends a running event — kill-switch, config limits, and WAF are the only brakes. That gap is a recorded operator risk acceptance.

## 4 · Assess data-breach duty (GDPR — operator is in Germany)

If personal data was plausibly read or exfiltrated: scope it — durable data subjects are **moderator accounts** (email, display name, country, locale); **transient event content within its 3-day window can contain participant personal data in real-names mode** (FR-4). The Art. 33 72-hour clock starts at *awareness of a breach*, not at suspicion — document your assessment and its time either way. From M12, follow the OPS-06 breach-response runbook (Art. 34 notification via admin export + external send); before M12, document, decide, and if in doubt consult the supervisory authority's guidance directly.

## 5 · Recover, verify, learn

1. Reverse the temporary levers (kill-switch off, WAF rules removed, banner cleared) once the incident is closed.
2. Verify: full `gitleaks git` history scan · GitHub security tab clean · Cloudflare + GitHub audit logs show only you · CI green · a real magic-link login round-trips on DEV.
3. Write the incident down: one summary line per action in `manual-config-log.md` (done as you went), plus — if the incident revealed a design gap — a decision-log entry and a tech-context amendment via the normal loop.
4. If the incident was user-visible: service notice (M6+) or a Discussions post (post-beta), calm and honest per the product's tone rules.

## Rotation blast-radius reference

| Credential | Rotating it breaks | Recovery path |
|---|---|---|
| `SESSION_SECRET` | every moderator session (logout) + participant cookies | users log in again / cookies re-mint — self-healing |
| `TURNSTILE_SECRET_KEY` | submissions/auth/creation for seconds between widget-rotate and secret-put | none needed |
| `CLOUDFLARE_API_TOKEN` | CI deploys | update both GitHub Environments |
| `ADMIN_EMAILS` | admin role grants at next login | set correct list, log in again |
| Access policy/sessions (M6) | admin UI access | Cloudflare dashboard is always sufficient to rebuild the wall (reset-by-construction requirement) |
