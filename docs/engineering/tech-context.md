# MicLine.app — Tech context

> Provenance: **P2 tech interview, 2026-07-14** (operator + Claude; Cloudflare docs re-verified same day) · **post-interview critical review, 2026-07-14** (operator notes + adversarial pass, operator-evaluated — amendment record in §23) · **additional-features review, 2026-07-15** (ten operator notes, operator-evaluated — amendment record in §24; product record: decision-log §AF). Authority: from Gate 2 on, **this file is the single HOW document** — every `/speckit.plan` reads it; `docs/inputs/tech-decisions.md` is provenance-only and is never cited by plans again. Product WHAT/WHY authority stays with `docs/product/*` (feature-inventory.md owns milestones/versions). Enduring principles live in `docs/product/constitution-input.md` → the constitution. If reality diverges from this file, update **this file** (and the decision log), never the input file.
>
> Conventions used here: "row 1–5" = the manual-step timeline in `docs/build-guide/02-setup.md §2.7` (row 5 = the M6 admin-wall row, added there by the AF review — Access path only; the password quick-start needs no console step). F-numbers = the Gate-2 work-package cut (briefs in `docs/product/feature-briefs/`). Facts marked **[verify @F00x]** appear again in §22.

## 1. Pinned stack

| Concern | Choice |
|---|---|
| Runtime | Cloudflare Workers, **one Worker** (per env), `nodejs_compat`, TypeScript strict (current stable TS) |
| Web framework | React Router **framework mode** + React 19, SSR, Vite + `@cloudflare/vite-plugin`. **RR major decided at F000 against the current official Cloudflare template** (v8 current as of 2026-07; v7 was planning-time default) — record the choice here at F000 **[verify @F000]** |
| API/WS routing | Hono mounted in the same Worker for `/api/*` + `/ws/*`; everything else falls through to the RR request handler |
| Realtime | One Durable Object per event (`EventRoom`), WebSocket **Hibernation API**, DO **SQLite** storage |
| Global DB | D1 + Drizzle ORM + drizzle-kit migrations (expand-first, always) |
| Per-event DB | DO SQLite via Drizzle `durable-sqlite` driver **[verify @F003]** |
| Cache/config | Workers KV: `CONFIG` namespace (platform config, flags, log level) + `code:⟨CODE⟩→eventId` hot map |
| Blob storage | R2 — **M7 only** (emergency-export snapshots); no bucket/binding before then |
| Email | Cloudflare **Email Service**, `send_email` binding (§14) |
| Bot defense | Turnstile, server-side siteverify (§15) |
| Rate limiting | **Two-tier**: Workers rate-limiting binding (10/60 s bursts) + D1/DO counters (long windows), one interface (§15) |
| Admin wall | **Dual-mode wall on `/admin*`** — `ADMIN_PASSWORD` quick-start or Cloudflare Access + one-time PIN (recommended; always wins); fails closed unconfigured; **the wall is the admin auth** (§11.4, AF-7) |
| Monorepo | pnpm 11 workspaces + Turborepo; Node 24 LTS via `.node-version`; exact pnpm via `packageManager` |
| Quality | Biome (lint+format) · Vitest + `@cloudflare/vitest-pool-workers` · Playwright later (M4+) |
| Styling | Tailwind CSS v4; design tokens centralized in `packages/ui` (M11 demand — no one-off styling) |
| IDs/codes | `crypto.randomUUID()` for entity ids **and the board capability token**; join codes = 6-char Crockford base32 (no I/L/O/U), case-insensitive input, collision-checked with retry; join URL `/e/⟨CODE⟩`, board URL `/b/⟨uuid⟩` |
| i18n | Lingui (@lingui/react + macros, PO catalogs), `en`+`de`; extraction + catalog compile inside `build` so unextracted strings fail CI **[verify @F000]** |

## 2. Architecture rules (binding on every plan)

1. **Service layer is the API.** All business logic lives in `worker/services/*` as plain functions `(env, …)`. Hono routes AND React Router loaders/actions call these directly — loaders never HTTP back into the same Worker.
2. **The DO is the single realtime authority.** It owns live state, serializes all mutations (single-threaded actor), broadcasts snapshot-then-deltas, and enforces the state-machine invariants (§8). Clients render server state; they never compute authoritative state. The edge (Worker) authenticates and passes identity/role via socket tags; the DO trusts tags for identity but **re-checks role on every mutating message**.
3. **Auth = magic links only; email is an identifier, never a credential** (§11). Admin access goes through the dedicated §11.4 wall **exclusively** — the wall is the admin auth; moderator auth plays no role on `/admin` and is never stacked with a second factor (AF-7).
4. **The line state machine is two-mode-shaped from the first schema** (§8): M9's pool/review/declined/merged states and curated ordering must slot in without a rewrite — and no curated machinery ships before M9.
5. **Entitlement seam (constitution article):** anything gated is checked only through one `can(actor, capability, ctx)` service; grants come from baseline + vouchers (M8) + admin, later possibly billing — gated code never knows the source. Anticipated from M1: `requireAdmin()` as the wall-guard seam (AF-7 — no `users.role` column; the admin is not a product user), append-only audit-log table (privileged admin/entitlement actions ONLY — REJ-09), soft-delete semantics, structured logs, ops-counter writes (§16, AF-3), design tokens only.
6. **Every external fact is re-verified at use time** (the CLAUDE.md Cloudflare-docs protocol); §22 lists what and when.
7. **One job + one test per lifetime:** every row of the product brief's data-lifecycle table maps to exactly one enforcement job and one test proving deletion happens (§10).
8. **Venue-NAT rule:** mutating participant actions are never rate-limit-keyed by bare IP; IP keys only on unauthenticated entry/read paths with room-scale ceilings (§15).
9. **Dependency budget:** every new runtime dependency needs an explicit justification in the plan that introduces it (constitution Article III — solo-operator maintenance burden); prefer Cloudflare primitives and the platform/standard library.

## 3. Environment model (DEV/PRD)

Full design: `docs/build-guide/03-github-operations.md` (binding for F000). Summary + interview confirmations:

| | `env.dev` | `env.production` |
|---|---|---|
| Worker (explicit `name` per env) | `micline-dev` | `micline` |
| Custom domain (auto-DNS on first deploy) | `dev.micline.app` | `micline.app` |
| D1 / KV / DO | own database, own `CONFIG` namespace, DOs isolated inside the Worker | own everything |
| `send_email` binding + sending domain | `dev.micline.app` — own onboarding, own SPF/DKIM/DMARC | `micline.app` — own onboarding |
| Cron triggers | **same schedule as production** (parity smoke-testing; Topic-0 decision) | lifecycle/purge sweeper |
| Deployed by | `deploy-dev.yml` — every merge to `main` (**main = DEV**) | `deploy-prd.yml` — every published Release, tag-checkout, after human approval |

Wrangler gotcha (verified): **bindings and vars are non-inheritable** — each env block is written out fully; `routes`/`triggers` inherit unless overridden; env-level explicit `name` avoids wrangler's `⟨name⟩-⟨env⟩` auto-naming.

**Secrets matrix (complete for current scope):**

| Secret | Store | When | Purpose |
|---|---|---|---|
| `CLOUDFLARE_API_TOKEN` + `CLOUDFLARE_ACCOUNT_ID` | GitHub Environments `dev` AND `production` (never repo-wide) | end of F000 (row 1) | pipelines |
| `SESSION_SECRET` | Wrangler secret ×2 envs + `.dev.vars` | before F002 (row 4) | HMAC for moderator session + participant cookies |
| `TURNSTILE_SECRET_KEY` | Wrangler ×2 + `.dev.vars` (local: test keys) | before F006 (rows 3+4) | siteverify |
| `FEEDBACK_EMAIL` | Wrangler ×2 | M4 / F012 | E-2 destination (admin-UI setting supersedes in M6) |
| `ADMIN_EMAILS` | Wrangler ×2 | M6 | Access-mode admin allowlist (§11.4, AF-7) |
| `ADMIN_PASSWORD` | Wrangler ×2 (optional) | M6 — password-mode wall only | admin-wall quick start (§11.4; inert once Access is configured) |

Plain (non-secret) vars: `TURNSTILE_SITE_KEY`, `EMAIL_FROM` (per env — PRD `no-reply@micline.app`, DEV `no-reply@dev.micline.app`; display name at M2 spec), `ENVIRONMENT` (`local`/`dev`/`production` — `local` exists ONLY in `.dev.vars`, never in `wrangler.jsonc`; it keys the §11.4 admin-wall bypass and the F001 auth stub), `HEALTH_ENDPOINT_ENABLED` (per env, F008 — deliberately a deploy-time var, not KV config: §16, AF-8), `ACCESS_TEAM_DOMAIN` + `ACCESS_APP_AUD` (M6, Access mode only — their presence selects Access mode, §11.4). Local: `.dev.vars` gitignored, `.dev.vars.example` committed with keys only.

**IaC posture** (02-setup §2.8): `wrangler.jsonc` is the IaC; D1/KV creation lives in `docs/runbooks/resource-bootstrap.md` (committed 2026-07-14 as a convention + command stub; F000 fills the real commands/ids); secrets are never IaC; residual console clicks = rows 1–4 **plus row 5 (M6, now the *recommended* path — AF-7): create Zero Trust org + Access application for `/admin` covering `micline.app` and `dev.micline.app`, one-time PIN IdP, allowlist policy** — the password quick-start alternative needs no console step (one `wrangler secret put ADMIN_PASSWORD`; signed 5-day session, default documented at the row) — logged in `manual-config-log.md` like everything else. R2 bucket creation joins the bootstrap runbook at M7.

**Resource naming & attribution (binding — the Cloudflare account is shared with other projects):** every account-level resource carries the `micline` name prefix — workers `micline`/`micline-dev`, D1 `micline`/`micline-dev`, KV `micline-config`/`micline-config-dev`, Turnstile widget `micline`, Access app `micline-admin` (M6), R2 buckets `micline-exports`/`micline-exports-dev` (M7), CI API token `micline-ci`. Additionally, every taggable resource gets **Resource Tags** `project:micline` + `env:dev|production` (Cloudflare Resource Tagging, public beta since 2026-04 — Workers/D1/KV/R2/zones; API-first, PUT replaces the whole tag set) for dashboard filtering and usage/billing attribution. Both conventions live in `docs/runbooks/resource-bootstrap.md`; tags are applied at resource creation and logged like any manual step. Honest limit: the Cloudflare invoice stays account-level — tags attribute usage, they don't split the bill.

## 4. Monorepo layout

```
apps/web/                 # the single deployable
  app/                    # React Router routes (SSR): /, /e/⟨CODE⟩, /b/⟨uuid⟩, console, /admin, marketing
  worker/                 # Hono /api+/ws, EventRoom DO, scheduled() cron handler
    services/             # THE API: plain (env, …) functions — events, codes, auth, email,
                          #   config, ratelimit, entitlements(M8), export(M5), purge
    emails/               # template registry: one module per email type (E-1…E-5)
    do/                   # EventRoom (state machine + presence + alarm multiplexer)
  CLAUDE.md               # nested agent context (F000)
packages/shared/          # zod schemas (HTTP + WS contracts + config schema), Drizzle schemas,
                          #   shared domain types — client and DO import the SAME contracts
packages/ui/              # design tokens + primitives (M11 demand; no one-off styling anywhere)
docs/ · specs/ · .specify/
```

## 5. Runtime architecture (one Worker, one request flow)

```
Browser ──▶ CF edge (zone micline.app) ──[M6: admin wall on /admin* — §11.4 (Access intercepts here if configured; in-Worker check either way)]──▶ Worker
  Hono middleware: security headers · structured logger (level from CONFIG) · cookie auth
  ├─ /api/*, /ws/*  → services → D1 · KV · EventRoom DO (WS upgrade tags: role, pid/uid)
  ├─ /e/⟨CODE⟩ /b/⟨uuid⟩ /admin /console /marketing → RR SSR loaders → the SAME services
  ├─ env.EMAIL.send() → Email Service (only via worker/services/email.ts + registry)
  ├─ Turnstile siteverify (M4) — server-side, on the three decided actions
  └─ scheduled() → cron sweeper (§10)
```

Board/participant/console all speak the same zod WS contracts from `packages/shared`. Position derivation on participant surfaces is client-side from the broadcast ordered line (visibility-filtered) — no extra server state.

## 6. Data model draft — **draft; the authoritative version emerges per-feature spec**

**D1 (global; strongly consistent; PITR ≤30 d — §10 honesty):**
- `users` — id, email (unique), display_name?, country?, console_locale, soft_deleted_at?, banned_at? (M6), created_at, last_active_at (inactivity purge input) — *no role column: the admin is not a product user (AF-7)*
- `events` — id, owner_id, **code** (unique), **board_token** (uuid, unique, indexed), title, starts_at/ends_at + original_starts_at/original_ends_at (±24 h anchor), status snapshot, settings (language, name_mode, profanity_filter, min/max question length, primer text + ack mode + confirmation statement; M9 pool/upvote/limit settings), **private_description** — *the whole row lives with the account* (Topic-6 decision), stats JSON (written at end; survives with row)
- `auth_tokens` — token_hash (pk), email, expires_at, created_at — **atomic single-use consume**; doubles as the per-email request counter
- `audit_log` — append-only; **privileged admin/entitlement actions only**; purged after the audit retention (platform config; default 12 mo)
- `aggregates` — anonymous, **no user/event keys by construction** (bucket, country, counts, durations); written as snapshot at event end; survives everything
- `ops_counters` — keyless operational counters (day bucket × counter key × count) behind one stats service (§16, AF-3); written from the milestone each signal first exists; retention = platform config (default 24 mo)
- M8: `codes` (incl. optional admin-set **locked email** — locked ⇒ single-use, redeemable only by that normalized address; PII with a redeem-by-bounded lifetime — AF-2), `grants` (wallet) · M6: ban/soft-delete columns + service-notice storage (M6 spec owns)

**EventRoom DO SQLite (per event; = "event content", self-purged end+3 d):**
- `questions` — id, pid, text, status, arrival_seq, order_key (M9 fractional index; unused in M3), timestamps, follow_up (M5); M9: review/decline (+reason), merge structure, votes
- `participants` — pid, display_name/nickname, joined_at, left_at?
- `presence` — connection bookkeeping via hibernation tags + grace deadlines
- `announcement` — the one current announcement + expires_at
- `schedule` — the alarm multiplexer's due-work rows (§10)

**KV:** `CONFIG` namespace — `platform:config` (one zod-validated JSON doc), `flag:event-creation-disabled`, `flag:maintenance-banner`, `flag:invite-only` (M8), `platform:log-level` · `code:⟨CODE⟩ → eventId` (write at creation, delete at event deletion; D1 fallback + backfill).

**Client:** `ml_session` (moderator, §11.2) · `ml_p_⟨eventId⟩` (participant, §11.3) · `ml_admin` (admin wall, password mode only, §11.4) · localStorage = explicit display-preference overrides only (§11.3).

## 7. Field-level data boundary (FR-5, decided)

**Event row bucket (D1, lives with the account):** code, board token, title, times + originals, all creation settings incl. primer + confirmation statement, private description, lifecycle metadata, stats. **Event content bucket (DO, purged end+3 d):** everything participant-derived (questions, names/nicknames, pids), decline reasons, votes, merge structure, follow-up flags, announcement. Privacy page states the split honestly (title/description/settings live with the account); F007 copy may advise against personal data in titles. Exports (M5) leave the deletion domain — responsibility line at export + privacy page.

## 8. The line state machine (two-mode-shaped; M3 ships open mode only)

Canonical statuses (full two-mode shape; **M3 uses only the bold path**):
`pending_review` (M9) → `pool` (M9) → **`queued` → `answered` | `skipped` | `withdrawn`** · plus M9 terminals `declined`, `merged` (into a primary).

- **Ordering:** M3 = arrival order, authoritative in the DO (`arrival_seq`); "Now speaking" = head of the line (derived, not a status — the ≤1-Now invariant is structural); "Up next" = second. M9 curated ordering uses a fractional `order_key` (O(1) insert/reorder, periodically renormalized) — the column exists from F003, no reorder machinery before M9.
- **Invariants (tested at the DO):** one active waiting item per pid (open mode; freed by answered/skipped/withdrawn) · position stability in open mode (your position only ever improves) · no silent reorder by anything · terminal immutability (10 s single-level undo for answered/skip is a designed exception window) · M9: votes advise, never reorder; one vote per (question, pid) best-effort; no self-referential merge cycles; merged items inherit the primary's outcome.
- **Visibility matrix (enforced in DO broadcast filtering):** participants see the full ordered line + Now; **skip-stub**: text removed from all public surfaces instantly, stub (attribution + "skipped") stays in the participant line, board never shows skipped items, owner keeps a chip; owner-only chips: own statuses (M9 adds in-pool/declined/merged); board never shows timestamps or vote counts; M9: hidden pool ⇒ pool items invisible to non-owners, upvotes auto-disabled.
- **Content rules at the submission boundary (SAFE-03/04, in F003):** trim → collapse whitespace → strip control chars (explicitly incl. bidi/direction-override characters — board-spoofing vector, AF-1) → platform 1–280 + per-event min/max (from F001 settings) → optional per-event wordlist profanity filter (default off; block-vs-mask semantics at spec, FR-5). All moderator-authored text renders inert everywhere (no auto-links, no images).
- **Announcements (MOD-02):** one current, replaces previous; bounded display lifetime (platform config, direction ~60 s) with uniform auto-expiry on all surfaces via the DO alarm; clear-early; re-send restarts.

## 9. Realtime design (EventRoom DO)

- **Hibernation API:** `ctx.acceptWebSocket(server, tags)` with tags `role:participant|moderator|board` + `pid:…`/`uid:…`; handlers `webSocketMessage/Close/Error`; per-socket `serializeAttachment` for connection context (small — limit noted **[verify @F003]**); runtime auto-answers pings without waking the DO; in-memory state is disposable by design — everything authoritative is in DO SQLite, constructor rehydrates lazily.
- **Protocol:** connect → full **snapshot** (role-filtered) → ordered **deltas**; every client action carries a client-generated idempotency id; the DO echoes authoritative results (server state wins on echo). Reconnect (auto, backoff) → fresh snapshot; own-identity restore via the pid cookie. This satisfies the **binding T9-2 criteria** (no silent loss · duplicate-free retries · explicit save states · automatic recovery) — verbatim acceptance criteria in the F003 spec.
- **Presence (FR-2, decided):** present = pid with ≥1 open WS (tag-counted). Disconnect starts the **reconnect grace** (platform config; **direction ~60 s**, final number at F003 spec); reconnect within grace keeps the slot; expiry frees it (alarm-driven). Admission: new pid rejected when `present + in-grace ≥ cap` → event-full state; known live/in-grace pid always re-attaches. Console's live count = this number. Accepted per FR-2: grace expiry at a full event ⇒ calm full screen on return.
- **Deploy behavior:** a production deploy restarts DOs — every socket drops once and reconnects into a fresh snapshot; state survives in DO SQLite. Reconnect logic is a **release-safety feature**; migrations expand-first so reconnecting clients from release N−1 tolerate release N schemas.
- **Per-DO soft throttle:** the DO sheds excess per-event load gracefully (calm retry states, never raw errors); DO soft limit ~1 000 req/s is far above the ~500-participant envelope.

## 10. Scheduling & purge architecture (decided)

- **The DO purges itself.** Each EventRoom multiplexes its **single alarm** (verified: exactly one per DO; official pattern) across a `schedule` table: announcement expiry → presence-grace expiries → auto-end broadcast (ends_at + 24 h) → **self-purge at end + 3 d** (`storage.deleteAll()` + alarm cleanup). Alarms wake idle/hibernated DOs, so purge fires without traffic. DO storage has **no time-travel** — content deletion is real.
- **Worker cron = D1-side enforcer + backstop** (same schedule both envs): expired `auth_tokens` cleanup · inactive-account purge (1 month; carve-outs: future event scheduled, M8 non-empty wallet) · `audit_log` > audit retention (platform config; default 12 mo) · `ops_counters` > counter retention (platform config; default 24 mo — AF-3) · expired unredeemed `codes` incl. any locked email (M8 — AF-2) · KV `code:*` removal on event deletion · **defensive sweep**: any event whose purge is overdue in D1 terms gets its DO re-armed (lost-alarm insurance).
- **Every lifetime = one job + one deletion-proving test** (F010 rule; M12 verifies/monitors, never introduces).
- **M7 seams (reserved now, built then):** cron + DO alarm handlers check the emergency flag before purging (overdue purges run 24 h after resume); the emergency **flag itself is NOT the KV config mechanism** — M7 spec owns its stronger-semantics design (freeze is push-driven through DOs; the flag blocks new activity). Emergency exports: pre-generated static files in **R2**, download endpoint `/exports/⟨random token⟩` (token + expiry in D1; validity = platform config, default ~24 h), **freeze-exempt, read-only, session-auth-free (the capability link is the auth), per-IP rate-limited**, R2 lifecycle auto-delete. Normal M5 exports are on-demand streams — nothing stored, hence no link validity there.
- **Config note:** every lifetime offset and link validity in this section is a platform-config value (§13) — the numbers shown are committed defaults, adjustable at runtime without redeploy (explicitly incl. audit-log retention and emergency-export link validity).

## 11. Auth & identity design (decided at interview)

### 11.1 Magic links (OQ7 — closed)
`POST /api/auth/request {email}` → Turnstile (from M4) → limits (§15 — the **per-email send cap ships with F002 itself**, 2026-07-14 review; the remaining matrix lands at F006) → **always 200, uniform timing** (send via `ctx.waitUntil`); banned/soft-deleted emails silently skip the send. Token: 32 random bytes → base64url in link; **SHA-256 hash stored in D1 `auth_tokens`** with `expires_at = now + 15 min`; **single-use enforced atomically** (`DELETE … RETURNING` on verify) — interview correction: D1, not KV, so replay is structurally impossible. Verify: expired/unknown ⇒ calm "request a new link" screen; success ⇒ user upserted (registration = first verify; M8 lockdown gates the *request* step), session issued. E-1 goes through the template registry (§14).

### 11.2 Moderator sessions
**Stateless** `ml_session` cookie: `uid.exp.HMAC-SHA256` (Web Crypto, `SESSION_SECRET`, constant-time compare), ~30 d, HttpOnly + Secure + SameSite=Lax. `requireUser()` = verify signature **and** load the D1 user row — ban/soft-delete/deletion bite on the next request despite statelessness. `requireAdmin()` is **not** part of moderator auth — it is the §11.4 wall guard; admin access never touches magic links or the `users` table (AF-7). No server-side session store; no "log out everywhere" (only `SESSION_SECRET` rotation — documented operator lever).

### 11.3 Participant identity (FR-5 — closed)
- **Per-event, no global identifier:** signed cookie `ml_p_⟨eventId⟩` = `{pid: random 128-bit, ack?: true}` + HMAC; minted lazily on first join. Cross-event correlation structurally impossible.
- **Lifetime (operator-corrected):** cookie `Max-Age` = current `ends_at` **+ 48 h** (covers the ±24 h reschedule envelope + auto-end), refreshed on every HTTP response; the 3-day purge window is a server-side concept and does **not** govern the cookie. DO participant rows die with the content purge.
- **Leave event** = clear cookie + DO marks session left (identity reset per B6/T1-2; prior questions keep their attribution snapshot; Turnstile re-runs on rejoin once M4 ships).
- **Ack flag lives only in the cookie** — the acknowledgment gate leaves zero server-side trace; leave/rejoin re-prompts the primer.
- **Names/nicknames live in the DO only** (event content) — never in the cookie; no PII on the wire per request.
- **Display prefs = localStorage, explicit overrides only (precedence rule, operator-ratified):** localStorage stores *only explicit user adjustments*, keyed per event + surface; every render computes the server-derived default fresh (I18N-02 derivation: participant surface = device language; board = event language) and applies an override only if one exists for this event+surface. Server-side changes therefore propagate unless deliberately overridden; no event inherits another's override. Edge cases (register-variant, dual-surface viewer, pre-start event-language change) resolve in the F012/I18N-02 spec against this rule.

### 11.4 Admin wall (OQ1 — closed 2026-07-14; **amended 2026-07-15, AF-7: dual-mode, and the wall IS the admin auth**)
- **The wall is the admin auth.** Admin access never uses product auth: no magic-link login, no `users` row, no role column — `requireAdmin()` = "this request passed the wall". The admin is not a product user (a separate ordinary moderator account exists if the operator also moderates). Audit-log actor: the Access-verified email, or `admin (password wall)`.
- **Mode selection — config-based, never header-sniffing, fixed precedence:** `ACCESS_TEAM_DOMAIN` + `ACCESS_APP_AUD` configured ⇒ **Access mode** (the password path is disabled — **Access always wins**) · else `ADMIN_PASSWORD` secret set ⇒ **password mode** · else ⇒ `/admin*` **fails closed with setup instructions** — a fresh or one-click deploy can never expose an unauthenticated admin UI (the product itself keeps running).
- **Access mode (recommended):** Access application covers `/admin*` on **both hostnames**; IdP = one-time PIN; policy = operator email allowlist. Zero Trust free tier covers this (1 seat needed; free-plan seat count historically 50 — **[verify @M6]**). **Defense in depth:** the Worker validates the Access JWT (`Cf-Access-Jwt-Assertion`) on every `/admin*` request — signature against the team JWKS, audience against `ACCESS_APP_AUD` — **and requires the JWT email ∈ `ADMIN_EMAILS`**; fails closed if absent/invalid. The wall is never dashboard-config-only. Note the PIN email comes from *Cloudflare*, so admin access never depends on MicLine's own email pipeline — deliberate (you need `/admin` exactly when auth/email is on fire).
- **Password mode (quick start):** `ADMIN_PASSWORD` (Wrangler secret) → login form at `/admin` → timing-safe verification, **throttled failures** (Tier-A per-IP burst + durable counter; numbers at the M6 spec) → signed `ml_admin` cookie (HMAC via `SESSION_SECRET` — rotating it also logs the admin out, same incident lever), **5-day validity (committed default — documented right next to the setup step: 02 §2.7 row 5 + `config-operations.md`)**, HttpOnly + Secure + SameSite=Strict, `/admin`-scoped. No email involved at all. The admin UI shows a standing nudge toward Access mode.
- **Reset story (the hard requirement — holds in both modes, by construction):** Access mode is managed entirely in the Cloudflare dashboard (zero deploys); password mode resets with one `pnpm wrangler secret put ADMIN_PASSWORD`. Either way: Cloudflare account access alone suffices, no lockout dead-end.
- Local dev: no wall locally — bypass + fake admin session exist only under `ENVIRONMENT=local` (a value set exclusively in gitignored `.dev.vars`; deployed envs run `dev`/`production` and can never take the bypass — deployed dev is walled too). *(S1 fix — the earlier `ENVIRONMENT=dev` guard would have opened the bypass on deployed DEV.)*
- Step-up on destructive actions (M7 ceremony owned by M7 spec; direction decided): fresh wall re-authentication in the active mode (Access re-auth, or password re-entry) + the product's typed confirmation.

## 12. Board capability token (closed)
`crypto.randomUUID()` (operator choice: UUID form) minted at event creation (F001), stored unique+indexed on the event row; route `/b/⟨uuid⟩`; page + board-WS upgrade validate it. Never derived from event id or join code. **Regenerate (M5):** overwrite in one write — old URL dies instantly and the DO kicks connected board sockets. Anti-enumeration: 122 random bits + the same calm "unknown" screen as unknown codes + per-IP lookup rate limit. No KV mirror (board lookups are rare; one indexed D1 read).

## 13. Platform config, flags, log level (OQ8 + FND-06 — closed)
- **KV `CONFIG` namespace**: `platform:config` = ONE JSON document validated by a zod schema in `packages/shared` that also defines **every committed default** (empty KV ⇒ working platform; stored values override; malformed config ⇒ defaults + `critical` log, never crash). Flags = individual keys (`flag:event-creation-disabled`, `flag:maintenance-banner`, `flag:invite-only` @M8). `platform:log-level` = own key.
- Read via one `config.ts` service with **~30 s in-isolate cache**; worst-case propagation ≤ ~90 s (KV edge + cache) — operator-accepted for every value incl. log level. DOs read through the same binding + cache. **Any UI that writes config or flags (the M6 admin UI) must tell the operator that a change can take up to ~90 s to take effect.**
- Catalog (initial): create-ahead window (14 d default, FR-3) · reschedule window ±24 h · participant caps (100/1 000) · event limits (1+1) · question-length platform bounds 1–280 · announcement lifetime (~60 s direction) · reconnect grace (~60 s direction) · purge offsets (content 3 d, auto-end +24 h, accounts 1 mo, audit 12 mo, resume grace 24 h) · ops-counter retention (24 mo default — AF-3) · text caps (confirmation statement, hide-message @M5, decline reason @M9) · export-link validity ~24 h (M7) · curated submission-limit default (M9). *(Deploy-time switches that must survive a KV failure are Wrangler vars, not config keys — sole current case: `HEALTH_ENDPOINT_ENABLED`, §16/AF-8.)*
- Operator surface: M1 = `docs/runbooks/config-operations.md` (created by F008 — one entry per key: exact `pnpm wrangler kv key put …` command, committed default, and description); M6 admin UI writes the same keys through the same service (with the ~90 s propagation notice above). **Every config key carries an operator-facing description** (zod `.describe()` in the shared schema), rendered wherever config is shown — runbook and admin UI — so the operator always knows exactly what a value does. **The M7 emergency-stop flag is explicitly NOT this mechanism** (§10).

## 14. Email architecture (registry decided; findings re-verified 2026-07-14)
- **Verified current:** binding key `send_email` (name e.g. `EMAIL`); API `env.EMAIL.send({from, to, subject, text, html, …})` → `{messageId}` (legacy EmailMessage/mimetext path exists; not needed). Limits: 5 MiB/message, ≤50 recipients. **Pricing published — Workers Paid: 3 000 sends/month included, then $0.35 per 1 000; not available on Free plan.** New accounts start with conservative auto-scaling daily quotas (harmless at magic-link volume). Local dev **simulates** sends (content written to local files — that's where the magic link is read in dev); `remote: true` on the binding opts into real sends from local. Domain onboarding (row 2) auto-creates SPF/DKIM/DMARC + bounce-subdomain MX **per sending domain**; domains onboard separately, and DEV sends from its own domain — PRD `micline.app`, DEV `dev.micline.app` (subdomain onboarding verified 2026-07-14; zone cap: 30 sending/routing domains combined). DEV experiments can never dent production sending reputation. **[verify @F002]**
- **Template registry (T8b-5):** one module per email type in `worker/emails/` — E-1 magic link (M2) · E-2 feedback (M4) · E-3 force-end (M6) · E-4/E-4b/E-5 emergency set (M7) — each exporting `subject/text/html(locale, params)`, every string through Lingui, rendered in the **moderator's console language**. `worker/services/email.ts` is the **only** send path (Article-II seam; fallback candidates if ever needed: Resend/Postmark behind the same seam + decision-log entry) and accepts **only registered templates** — ad-hoc sends are impossible by construction. CI renders every registered template in every locale.
- **react.email: evaluated, not adopted** (interview decision): plain-text-first + one shared minimal HTML layout (inline styles, no images, no tracking) serves six short transactional mails; the registry's render seam keeps react.email a drop-in if M11 wants designed mail.
- Sender: `EMAIL_FROM` var per env — `no-reply@micline.app` (PRD) / `no-reply@dev.micline.app` (DEV); display name at M2 spec.

## 15. Abuse & hardening stack
- **Turnstile (M4, SAFE-01):** invisible widget on exactly three actions — auth request, event creation, first submission per participant session (re-run after leave/rejoin). One widget, both hostnames (free plan: 20 widgets × 10 hostnames — fits). Server-side siteverify from the Worker; local dev + CI use Cloudflare's documented test keys (always-pass/always-fail/token-spent). **[verify @F006]**
- **Rate limits (SAFE-02 — keys + mechanisms decided; numbers at F006 spec, revisited at the OQ6 post-tech limit review):** one interface `worker/services/ratelimit.ts`; **Tier A** = Workers rate-limiting binding (verified: 10 s or 60 s windows only, per-colo, approximate/permissive — burst brake, never an accountant); **Tier B** = durable counters (D1 for account-scoped: magic-link per-email via `auth_tokens` count, event-creation per-uid daily, reschedule per-event daily, feedback per-uid daily; DO for event-scoped: per-sid submission bursts, per-event ceilings/soft throttle). Matrix: magic-link = per-email primary (**ships with F002** — 2026-07-14 review; number at its spec) + generous per-IP burst · creation = per-uid · **reschedule = 60 s brake + per-event daily cap** (T8-5) · submission = per-sid + one-active-item (the real limiter) + per-event DO ceiling · join/board lookups + WS upgrades = per-IP with **room-scale ceilings** (venue-NAT rule §2.8; Cloudflare's DDoS layer fronts everything).
- **Headers & transport (F000 baseline, tightened F007):** HSTS (prod), nosniff, `frame-ancestors 'none'`, `Referrer-Policy: no-referrer` (capability-URL hygiene — AF-1), CSP report-only → enforced at F007; **search-engine indexing is default-deny (AF-6): `X-Robots-Tag: noindex` on every response in both environments from F000 — F007 explicitly allowlists the marketing surfaces on production and ships robots.txt + sitemap.xml (marketing pages only); DEV is never indexable**; WS upgrades require an allow-listed Origin **per environment** (production ⇒ `micline.app` only · dev ⇒ `dev.micline.app` only · local ⇒ localhost) — checked before any auth work.
- **Input-handling addenda (AF-1):** request-body size caps on API routes before schema parsing (F000 middleware) · mutating routes verify same-origin signals (Origin/Sec-Fetch-Site) on top of SameSite cookies — an explicit CSRF stance, not an accident · the M6 admin password login is throttled per §11.4.
- **Export hardening (M5, AF-1):** exports render user content inert like every surface — CSV fields formula-escaped, the Line snapshot (HTML) escapes all user content and contains no executable script; exact rules at the M5 spec (§20).
- **Content rules:** §8. **Threat map:** file 03 §G (one-screen table) stays authoritative for pipeline threats.
- **Incident response & residual risks:** operator playbook = `docs/runbooks/security-incident-response.md` (contain → rotate → escalate; `SESSION_SECRET` rotation = the designed global-logout lever). Accepted residual risks (assessed 2026-07-14; +AF 2026-07-15): mailbox = identity — inherent to magic links, so the **operator mailbox needs strong 2FA** (it also anchors the Access-mode PIN and `ADMIN_EMAILS`); no per-session revocation (rotation is global); the password-mode admin wall is a single shared secret — accepted for quick start, mitigated by timing-safe checks + throttling + the standing Access nudge (AF-7); board capability URL leakable until M5 regeneration exists; the FR-1 beta window; zero-gate viewing with a code (product pillar); public repo — attackers read the code, so nothing may rely on obscurity.

## 16. Observability, logging & admin statistics
Structured JSON logs (one logger, request-id + event-id correlation) → Workers Logs; global level from `platform:log-level` (debug ↔ critical, ≤ ~90 s propagation, FND-06 closed). No third-party sinks in current scope; the M12 OPS-01 decision picks the long-term pipeline against measured needs. Metrics visibility ladder (T9-4): participants none · moderators in-event count only · admin metrics wall (M6, own counters) · raw infra operator-only.

- **Admin-stats data model (AF-3) — three stores:** per-event `events.stats` snapshot (recap; dies with the event row) · anonymous `aggregates` (survive indefinitely, unlinkable by construction) · keyless **`ops_counters`** (day bucket × counter key; one stats service in `worker/services/`; retention = platform config, default 24 mo). Counters are written **from the milestone each signal first exists**, so the M6 metrics wall opens with history: **F001/F003** event lifecycle (created / started / ended manually / auto-ended / deleted) + **event-full rejections** (the OQ6 capacity evidence) + peak concurrent presence · **F002** auth funnel (links requested / consumed / expired) + emails per type + send failures · **F006** Turnstile failures + rate-limit hits per limiter · **F010** purge-job runs + rows purged per lifetime (the M12 verification evidence) · **F012** feedback count · **M6** force-end / soft-delete / ban counts · **M8** codes created/redeemed + active-grant totals. Everything is a keyless count — constitution article V holds. ADM-05 reads these; the M6 usage/cost panel reads Cloudflare's API (unchanged); € projection arrives with M12 OPS-02.
- **Health endpoint (FND-07, in F008 — AF-8):** `GET /health`, enabled per environment by the deploy-time var `HEALTH_ENDPOINT_ENABLED` (a Wrangler var by design — must work when KV is the failure), **never behind auth**; 200 only when D1 (`SELECT 1`) and KV (known-key read) respond, else 503; minimal body (no versions, no internals), `Cache-Control: no-store`; per-IP burst ceiling generous for 1/min uptime monitors. Reports **infrastructure** health — deliberately truthful-200 during an M7 emergency stop (intentional quiescence ≠ outage; the M7 spec references this). Uptime monitoring (build-guide 10 §C seed) points here.

## 17. CI/CD & delivery
Five required checks (`lint` `typecheck` `test` `build` `secrets`) + CodeQL/Dependabot/push-protection as background gates; deploys ONLY via Actions (merge→DEV; published Release→PRD with tag checkout + human approval; prereleases allowed; patch releases defects-only). Workflow hardening: SHA-pinned actions, top-level `permissions: contents: read`, no untrusted-text interpolation, Environment-scoped secrets. Migrations: `wrangler d1 migrations apply --env ⟨e⟩ --remote` in the pipelines only; local = miniflare. **Expand-first schema rule** (§9 deploy behavior). Full lifecycle + release mechanics: file 03 §D/§F.

## 18. Test strategy
- **Unit (Vitest, workers pool):** every state-machine transition + invariant (F003 starts the **≥80 % service/DO coverage gate**), auth flows (token consume atomicity, cookie tampering, anti-enumeration timing), config-schema defaults/fallback, email templates × locales, rate-limit interface.
- **Integration:** two scripted WS clients through the full M3 script (create→open→join→submit "saved as #N"→second-submit blocked→skip+stub→undo→answered auto-pull→you're-up→withdraw→announcement expiry→reconnect-snapshot→end); presence/grace admission tests; **deletion-proving tests per lifetime** (F010).
- **E2E (Playwright, from M4):** the release-gate smoke scripts (file 06) automated where cheap; two-device flows stay manual smokes.
- Miniflare simulates D1/KV/DO/email locally (miniflare is maintained inside the `cloudflare/workers-sdk` monorepo — the standalone `cloudflare/miniflare` repo is archived); cron handlers testable via the local scheduled endpoint. Turnstile: test keys.
- **Local inspection:** Cloudflare **Local Explorer** — browser UI at `/cdn-cgi/explorer` on the local dev server (on by default with Vite plugin ≥ 1.32 / Wrangler ≥ 4.82) — browses and edits local KV, D1, and DO-SQLite state (R2 later). Prefer it over ad-hoc dump code when debugging local state.

## 19. Cloudflare product facts (verified 2026-07-14 — re-verify per §22 at use time)
| Product | Facts that shaped decisions |
|---|---|
| Workers Paid | $5/mo; 10 M req + 30 M CPU-ms incl.; unlocks DO/D1/KV allowances; cron free (15-min CPU cap/invocation). Free plan: 100 k req/day (dev-only relevance) |
| Durable Objects | SQLite-backed on Free + Paid; 10 GB/object (Paid); ~1 000 req/s soft/DO; hibernation billing pause; auto ping/pong without wake; **exactly one alarm/DO**, at-least-once, ~6 retries w/ backoff, wakes hibernated DOs; in-memory state resets on hibernation |
| D1 | 10 GB/db (Paid); 1 000 queries/invocation; 100 bound params/query; **Time Travel always-on, non-disableable: 30 d Paid / 7 d Free** |
| KV | 25 MiB values; 1 write/s per key; eventually consistent (≤ ~60 s global); paid = effectively unmetered at our scale |
| Email Service | §14 — pricing published (3 000 incl. + $0.35/1k, Paid only); `send_email` binding confirmed; simulated local sends; per-domain onboarding (subdomains onboard separately — DEV sends from `dev.micline.app`) |
| Turnstile | Free: 20 widgets × 10 hostnames; unlimited (site)verify; documented test keys; invisible mode available |
| Rate-limit binding | 10/60 s periods only; per-colo; approximate/permissive (⇒ two-tier design §15) |
| Access (Zero Trust) | One-time PIN IdP; seat consumed per authenticated user; free tier sufficient for 1 admin (seat count **[verify @M6]**); JWT assertion validatable in-Worker |
| Wrangler | bindings/vars non-inheritable per env; env-level `name` override; per-env cron triggers; local scheduled-handler test route |
| Resource Tagging | Public beta (since 2026-04): key-value tags on Workers/D1/KV/R2/zones; filter + attribute usage per project/env; API-first (PUT replaces the full tag set) — feeds the §3 naming/tagging rule |

Running-cost picture (typical scenarios, EUR): `docs/engineering/runcost-estimate.md` (list-price estimate, 2026-07-14; superseded by the M12 OPS-02 measured cost model when that exists).

## 20. Decisions owned by later specs (recorded, not dropped)
| Decision | Owner |
|---|---|
| Exact numeric rate limits + per-DO throttle values (the magic-link **per-email cap** number lands earlier, at the F002 spec — pulled forward 2026-07-14) | F006 spec (+ OQ6 limit revisit before the M4 beta gate) |
| Reconnect-grace final number (direction 60 s) | F003 spec |
| Sender display name; magic-link screen copy; anti-enumeration timing details | F002 / M2 spec |
| Nickname uniqueness per event + generation language (direction: event language); profanity block-vs-mask + wordlist source | F003/F005 specs (FR-5) |
| Failure-state copy (closed list), locale edges (register-variant, dual-surface) | F012 spec |
| PITR-honesty sentence wording (privacy page + deletion UI) | F007 / F011 specs |
| Export injection-hardening rules — CSV formula-escape set, HTML-snapshot escaping (AF-1) | M5 spec |
| Admin-wall integration details for **both modes** (password login UX + throttle values + Access-nudge copy + per-mode step-up re-auth; JWKS validation pattern), force-end wording, consequence previews | M6 spec (AF-7) |
| Voucher email-lock UX — set/display, mismatch copy, expired-locked-code purge (AF-2) | M8 kickoff |
| Self-hosting paths — deploy-button contract, C3 `--template` behavior, deploy-shaped config without operator domains, `SELF_HOSTING` guide (AF-9) | M12 / OPS-07 |
| Emergency flag semantics, stop/export ceremony, E-4/E-4b/E-5 + emergency screens, R2 bucket + `/exports/⟨token⟩` details | M7 spec |
| Voucher UX (OQ3), god-voucher presentation, clock defaults | M8 kickoff |
| Filter matrix, curated defaults, submission-limit default/range, pool-collapse defaults, co-mod link mechanism + powers matrix | M9 kickoff |
| Markdown-lite subset, TMS-vs-Lunaria (OQ4), de-formal implementation | M10 kickoff |
| Logo/colors (OQ9), sound cue, board appearance | M11 kickoff |
| Observability pipeline, cost formulas, degraded modes (post-load-test), IaC re-open | M12 evidence gate |

## 21. Corrections applied vs docs/inputs/tech-decisions.md (provenance)
1. Email pricing **published** (was "not final"); binding/API/local-dev confirmed; fallback seam kept as discipline, urgency downgraded.
2. Rate-limit binding narrower than assumed (10/60 s, per-colo) ⇒ two-tier design.
3. Magic-link token store moved **KV → D1** (atomic single-use).
4. Question-status vocabulary corrected to product semantics (`pending→pool→queued→speaking→answered/dismissed` sketch superseded by §8; "dismissed" does not exist; skip-stub visibility replaces "participants never see others' dismissed"; votes/review are M9-only; LINE-08 added).
5. "Retention enforcement may ship M5" superseded by P3-6 (ships M4).
6. `linePosition non-null iff queued` deferred to M9 (M3 = arrival order; column reserved).
7. Board token = UUID (operator), not opaque non-UUID string.
8. Old milestone labels (M1/M2/M2A/M2B/M5) mapped to the P3 M1–M13 lineup throughout.
9. R2 added to the stack (M7 only).
10. Access chosen for the admin wall (was open).
11. KV min-TTL/cacheTtl fine print noted for F002-era uses **[verify @F002]** — no design depends on sub-minute KV expiry.

## 22. Verify at implementation time (every external fact)
- **F000:** current `@cloudflare/vite-plugin` scaffold + official RR template (pick the RR major, record in §1) · Lingui × chosen-RR-major × Vite integration + catalog-in-build wiring · wrangler env config shape (non-inheritance, custom domains, per-env crons) · gitleaks/CodeQL/Dependabot setup currency · pnpm/Node current pins · miniflare currency via `cloudflare/workers-sdk` (standalone repo archived) · Resource Tagging API (beta) when tagging the created resources.
- **F008:** KV binding API + `wrangler kv key put` syntax; min `expirationTtl`/`cacheTtl` values.
- **F001:** D1 migrations commands per env · Drizzle/D1 driver currency · QR generation lib choice.
- **F002:** exact `send_email` wrangler shape + `send()` signature + local-dev file paths (fetch email-service/llms.txt again) · `dev.micline.app` subdomain onboarding steps (email-service "subdomains" page) · Email quotas for a new account · KV fine print if any KV use lands here.
- **F003:** DO hibernation API names + serializeAttachment size limit · alarm retry semantics · `durable-sqlite` Drizzle driver status · WS message-size limits · vitest-pool-workers DO testing patterns.
- **F006:** rate-limit binding config shape/status · Turnstile siteverify contract + current test keys.
- **F010:** cron trigger syntax + local testing route · D1 batch/param limits for purge jobs.
- **M6:** Zero Trust free-plan seat count + Access app/policy setup + JWT validation pattern incl. JWKS/AUD specifics (fetch cloudflare-one docs) — then log row 5 (Access mode only; password mode needs no external verification).
- **M7:** R2 bucket + binding + lifecycle rules + multipart/streaming write patterns; D1 Time-Travel retention re-check for the honesty copy.
- **M12:** Deploy-to-Cloudflare button contract (monorepo/config-path support, resource provisioning) + `npm create cloudflare -- --template` behavior for OPS-07 (AF-9).
- **Ongoing:** any Cloudflare beta-status change for Email Service; Workers pricing table at each milestone kickoff (02-setup §2.9 ritual + `docs/runbooks/maintenance.md`).

## 23. Post-interview review amendments (2026-07-14)

Same-day critical review of the tech-interview artifacts (operator notes + adversarial pass; prompt preserved in build-guide file 04, "Tech interview artifact review"). Every finding below was operator-evaluated; all listed items were **accepted**:

1. **S1 (security fix):** `ENVIRONMENT` becomes `local`/`dev`/`production`; the admin-wall bypass + F001 auth stub key on `local` only, which exists solely in `.dev.vars` (§3, §11.4). The previous `ENVIRONMENT=dev` guard would have opened the bypass on deployed DEV.
2. **S2 (security fix):** the magic-link **per-email send cap** ships with F002 instead of F006 — closes the M2→M4 window in which the live-but-unannounced PRD auth endpoint had no Turnstile and no rate limits (§11.1, §15, §20; F002/F006 briefs; inventory AUTH-01/SAFE-02 notes).
3. **S3 (security fix):** WS Origin allowlist is per environment; localhost only locally (§15).
4. **Email domains split:** DEV sends from `dev.micline.app`, PRD from `micline.app` — separate onboardings, per-env `EMAIL_FROM` (§3, §14, §19; build-guide 02 §2.7 row 2; F002 brief).
5. **Resource naming + tagging rule** for the shared Cloudflare account: `micline` name prefix everywhere + Resource Tags `project:micline`, `env:*` (§3, §19; `docs/runbooks/resource-bootstrap.md`).
6. **Config transparency:** every config key carries an operator-facing description; the M6 admin UI must show the ≤ ~90 s propagation notice after writes; audit-log retention and emergency-export link validity confirmed **runtime**-configurable (superset of the requested deploy-time configurability) and annotated as such (§10, §13; F008 brief; inventory ADM-07 note). Normal M5 exports stream on demand — no stored link, hence no validity window there.
7. **New committed docs:** `docs/engineering/runcost-estimate.md` (typical-scenario EUR run costs) · `docs/runbooks/security-incident-response.md` (operator incident playbook incl. credential rotation) · `docs/runbooks/maintenance.md` (recurring checks) · `docs/runbooks/resource-bootstrap.md` (stub — F000 completes it).
8. **Process rules:** §2 rule 9 (dependency budget) · test-per-feature rule added to constitution-input IX and CONTRIBUTING · Local Explorer + workers-sdk/miniflare notes (§18, §22).

Residual-risk register: §15. Assessment verdict: the design is sound — no unmitigated bad-actor path was identified beyond the explicitly recorded acceptances.

## 24. Additional-features review amendments (2026-07-15)

Second review pass: the operator's ten feature/tweak notes, challenged one by one before the P3 constitution (prompt preserved in build-guide file 04, "Introduction of additional features + critical review"; **product-level record: decision-log §AF** — cite AF-numbers for provenance). Every item was operator-evaluated; all listed were **accepted**:

1. **AF-7 — admin wall dual-mode; the wall IS the admin auth** (supersedes the pure-Access OQ1 closure in §23-era §11.4): `ADMIN_PASSWORD` quick-start (signed 5-day session, timing-safe, throttled) or Cloudflare Access + one-time PIN (recommended; configuring it disables the password path — Access always wins); config-based mode selection; fails closed with setup instructions when unconfigured; admin access never uses product auth — **`users.role` dropped** (no consumer). Updated: §1, §2 (rules 3+5), §3 (secrets/vars/row 5), §6, §11.2, §11.4, §20, §22.
2. **AF-8 — `/health` endpoint (new FND-07, M1/F008):** deploy-time var (not KV), D1+KV checks, auth-free, infrastructure-health semantics (§3, §13, §16).
3. **AF-1 — hardening addenda:** export injection rules (M5 — CSV formula-escape, HTML-snapshot escaping), request-body size caps, explicit CSRF stance, bidi/direction-override strip in SAFE-03, `Referrer-Policy: no-referrer` (§8, §15, §20).
4. **AF-5/AF-6 — SEO + default-deny indexing:** `X-Robots-Tag: noindex` everywhere from F000, both envs; F007 allowlists marketing surfaces on PRD and owns the SEO checklist + robots.txt + sitemap (§15; F007 brief).
5. **AF-3 — admin-stats model:** three stores incl. the new keyless `ops_counters` (day-bucketed, 24-mo config retention), counters written from each signal's introducing milestone so the M6 wall opens with history (§6, §10, §13, §16; ADM-05).
6. **AF-2 — voucher email lock (M8):** optional admin-set lock; locked ⇒ single-use, redeemable only by that address; locked email is PII purged with the expired code (§6, §10, §20; ENT-03).
7. **AF-9 — self-hosting at M12 (new OPS-07):** Deploy-to-Cloudflare button + C3 `--template` scaffold + `SELF_HOSTING` guide; forward-compat demands binding on M1–M4: no hard-coded hostnames outside config, deploy-shaped config path, `/health`; honest one-click semantics — the AF-7 fail-closed wall is the backstop (§20, §22).
8. **AF-4/AF-10 — process:** intake funnel (build-guide 05 §7.0 — idea → decision-log → inventory → brief → loop, both build-time and post-`v1.0.0`); documentation map at `docs/README.md` (authority rules; linked from root README + CLAUDE.md).
