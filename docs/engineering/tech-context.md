# MicLine.app — Tech context

> Provenance: **P2 tech interview, 2026-07-14** (operator + Claude; Cloudflare docs re-verified same day). Authority: from Gate 2 on, **this file is the single HOW document** — every `/speckit.plan` reads it; `docs/inputs/tech-decisions.md` is provenance-only and is never cited by plans again. Product WHAT/WHY authority stays with `docs/product/*` (feature-inventory.md owns milestones/versions). Enduring principles live in `docs/product/constitution-input.md` → the constitution. If reality diverges from this file, update **this file** (and the decision log), never the input file.
>
> Conventions used here: "row 1–5" = the manual-step timeline in `docs/build-guide/02-setup.md §2.7` (+ row 5 added below). F-numbers = the Gate-2 work-package cut (briefs in `docs/product/feature-briefs/`). Facts marked **[verify @F00x]** appear again in §22.

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
| Admin wall | **Cloudflare Access + one-time PIN** on `/admin*`, both hostnames (§11.4) |
| Monorepo | pnpm 11 workspaces + Turborepo; Node 24 LTS via `.node-version`; exact pnpm via `packageManager` |
| Quality | Biome (lint+format) · Vitest + `@cloudflare/vitest-pool-workers` · Playwright later (M4+) |
| Styling | Tailwind CSS v4; design tokens centralized in `packages/ui` (M11 demand — no one-off styling) |
| IDs/codes | `crypto.randomUUID()` for entity ids **and the board capability token**; join codes = 6-char Crockford base32 (no I/L/O/U), case-insensitive input, collision-checked with retry; join URL `/e/⟨CODE⟩`, board URL `/b/⟨uuid⟩` |
| i18n | Lingui (@lingui/react + macros, PO catalogs), `en`+`de`; extraction + catalog compile inside `build` so unextracted strings fail CI **[verify @F000]** |

## 2. Architecture rules (binding on every plan)

1. **Service layer is the API.** All business logic lives in `worker/services/*` as plain functions `(env, …)`. Hono routes AND React Router loaders/actions call these directly — loaders never HTTP back into the same Worker.
2. **The DO is the single realtime authority.** It owns live state, serializes all mutations (single-threaded actor), broadcasts snapshot-then-deltas, and enforces the state-machine invariants (§8). Clients render server state; they never compute authoritative state. The edge (Worker) authenticates and passes identity/role via socket tags; the DO trusts tags for identity but **re-checks role on every mutating message**.
3. **Auth = magic links only; email is an identifier, never a credential** (§11). Admin access adds the dedicated Access wall in front — never a factor stacked onto moderator auth.
4. **The line state machine is two-mode-shaped from the first schema** (§8): M9's pool/review/declined/merged states and curated ordering must slot in without a rewrite — and no curated machinery ships before M9.
5. **Entitlement seam (constitution article):** anything gated is checked only through one `can(actor, capability, ctx)` service; grants come from baseline + vouchers (M8) + admin, later possibly billing — gated code never knows the source. Anticipated from M1: `users.role`, `requireAdmin()`, append-only audit-log table (privileged admin/entitlement actions ONLY — REJ-09), soft-delete semantics, structured logs, design tokens only.
6. **Every external fact is re-verified at use time** (the CLAUDE.md Cloudflare-docs protocol); §22 lists what and when.
7. **One job + one test per lifetime:** every row of the product brief's data-lifecycle table maps to exactly one enforcement job and one test proving deletion happens (§10).
8. **Venue-NAT rule:** mutating participant actions are never rate-limit-keyed by bare IP; IP keys only on unauthenticated entry/read paths with room-scale ceilings (§15).

## 3. Environment model (DEV/PRD)

Full design: `docs/build-guide/03-github-operations.md` (binding for F000). Summary + interview confirmations:

| | `env.dev` | `env.production` |
|---|---|---|
| Worker (explicit `name` per env) | `micline-dev` | `micline` |
| Custom domain (auto-DNS on first deploy) | `dev.micline.app` | `micline.app` |
| D1 / KV / DO | own database, own `CONFIG` namespace, DOs isolated inside the Worker | own everything |
| `send_email` binding | same onboarded `micline.app` sending domain | same |
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
| `ADMIN_EMAILS` | Wrangler ×2 | M6 | admin role bootstrap (ADM-01) |

Plain (non-secret) vars: `TURNSTILE_SITE_KEY`, `EMAIL_FROM` (default `no-reply@micline.app`; display name at M2 spec), `ENVIRONMENT` (`dev`/`production`). Local: `.dev.vars` gitignored, `.dev.vars.example` committed with keys only.

**IaC posture** (02-setup §2.8): `wrangler.jsonc` is the IaC; D1/KV creation lives in committed `docs/runbooks/resource-bootstrap.md`; secrets are never IaC; residual console clicks = rows 1–4 **plus new row 5 (M6): create Zero Trust org + Access application for `/admin` covering `micline.app` and `dev.micline.app`, one-time PIN IdP, allowlist policy** — logged in `manual-config-log.md` like everything else. R2 bucket creation joins the bootstrap runbook at M7.

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
Browser ──▶ CF edge (zone micline.app) ──[M6: Access wall intercepts /admin*]──▶ Worker
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
- `users` — id, email (unique), display_name?, country?, console_locale, role ('user'|'admin'), soft_deleted_at?, banned_at? (M6), created_at, last_active_at (inactivity purge input)
- `events` — id, owner_id, **code** (unique), **board_token** (uuid, unique, indexed), title, starts_at/ends_at + original_starts_at/original_ends_at (±24 h anchor), status snapshot, settings (language, name_mode, profanity_filter, min/max question length, primer text + ack mode + confirmation statement; M9 pool/upvote/limit settings), **private_description** — *the whole row lives with the account* (Topic-6 decision), stats JSON (written at end; survives with row)
- `auth_tokens` — token_hash (pk), email, expires_at, created_at — **atomic single-use consume**; doubles as the per-email request counter
- `audit_log` — append-only; **privileged admin/entitlement actions only**; 12-month purge
- `aggregates` — anonymous, **no user/event keys by construction** (bucket, country, counts, durations); written as snapshot at event end; survives everything
- M8: `codes`, `grants` (wallet) · M6: ban/soft-delete columns + service-notice storage (M6 spec owns)

**EventRoom DO SQLite (per event; = "event content", self-purged end+3 d):**
- `questions` — id, pid, text, status, arrival_seq, order_key (M9 fractional index; unused in M3), timestamps, follow_up (M5); M9: review/decline (+reason), merge structure, votes
- `participants` — pid, display_name/nickname, joined_at, left_at?
- `presence` — connection bookkeeping via hibernation tags + grace deadlines
- `announcement` — the one current announcement + expires_at
- `schedule` — the alarm multiplexer's due-work rows (§10)

**KV:** `CONFIG` namespace — `platform:config` (one zod-validated JSON doc), `flag:event-creation-disabled`, `flag:maintenance-banner`, `flag:invite-only` (M8), `platform:log-level` · `code:⟨CODE⟩ → eventId` (write at creation, delete at event deletion; D1 fallback + backfill).

**Client:** `ml_session` (moderator, §11.2) · `ml_p_⟨eventId⟩` (participant, §11.3) · localStorage = explicit display-preference overrides only (§11.3).

## 7. Field-level data boundary (FR-5, decided)

**Event row bucket (D1, lives with the account):** code, board token, title, times + originals, all creation settings incl. primer + confirmation statement, private description, lifecycle metadata, stats. **Event content bucket (DO, purged end+3 d):** everything participant-derived (questions, names/nicknames, pids), decline reasons, votes, merge structure, follow-up flags, announcement. Privacy page states the split honestly (title/description/settings live with the account); F007 copy may advise against personal data in titles. Exports (M5) leave the deletion domain — responsibility line at export + privacy page.

## 8. The line state machine (two-mode-shaped; M3 ships open mode only)

Canonical statuses (full two-mode shape; **M3 uses only the bold path**):
`pending_review` (M9) → `pool` (M9) → **`queued` → `answered` | `skipped` | `withdrawn`** · plus M9 terminals `declined`, `merged` (into a primary).

- **Ordering:** M3 = arrival order, authoritative in the DO (`arrival_seq`); "Now speaking" = head of the line (derived, not a status — the ≤1-Now invariant is structural); "Up next" = second. M9 curated ordering uses a fractional `order_key` (O(1) insert/reorder, periodically renormalized) — the column exists from F003, no reorder machinery before M9.
- **Invariants (tested at the DO):** one active waiting item per pid (open mode; freed by answered/skipped/withdrawn) · position stability in open mode (your position only ever improves) · no silent reorder by anything · terminal immutability (10 s single-level undo for answered/skip is a designed exception window) · M9: votes advise, never reorder; one vote per (question, pid) best-effort; no self-referential merge cycles; merged items inherit the primary's outcome.
- **Visibility matrix (enforced in DO broadcast filtering):** participants see the full ordered line + Now; **skip-stub**: text removed from all public surfaces instantly, stub (attribution + "skipped") stays in the participant line, board never shows skipped items, owner keeps a chip; owner-only chips: own statuses (M9 adds in-pool/declined/merged); board never shows timestamps or vote counts; M9: hidden pool ⇒ pool items invisible to non-owners, upvotes auto-disabled.
- **Content rules at the submission boundary (SAFE-03/04, in F003):** trim → collapse whitespace → strip control chars → platform 1–280 + per-event min/max (from F001 settings) → optional per-event wordlist profanity filter (default off; block-vs-mask semantics at spec, FR-5). All moderator-authored text renders inert everywhere (no auto-links, no images).
- **Announcements (MOD-02):** one current, replaces previous; bounded display lifetime (platform config, direction ~60 s) with uniform auto-expiry on all surfaces via the DO alarm; clear-early; re-send restarts.

## 9. Realtime design (EventRoom DO)

- **Hibernation API:** `ctx.acceptWebSocket(server, tags)` with tags `role:participant|moderator|board` + `pid:…`/`uid:…`; handlers `webSocketMessage/Close/Error`; per-socket `serializeAttachment` for connection context (small — limit noted **[verify @F003]**); runtime auto-answers pings without waking the DO; in-memory state is disposable by design — everything authoritative is in DO SQLite, constructor rehydrates lazily.
- **Protocol:** connect → full **snapshot** (role-filtered) → ordered **deltas**; every client action carries a client-generated idempotency id; the DO echoes authoritative results (server state wins on echo). Reconnect (auto, backoff) → fresh snapshot; own-identity restore via the pid cookie. This satisfies the **binding T9-2 criteria** (no silent loss · duplicate-free retries · explicit save states · automatic recovery) — verbatim acceptance criteria in the F003 spec.
- **Presence (FR-2, decided):** present = pid with ≥1 open WS (tag-counted). Disconnect starts the **reconnect grace** (platform config; **direction ~60 s**, final number at F003 spec); reconnect within grace keeps the slot; expiry frees it (alarm-driven). Admission: new pid rejected when `present + in-grace ≥ cap` → event-full state; known live/in-grace pid always re-attaches. Console's live count = this number. Accepted per FR-2: grace expiry at a full event ⇒ calm full screen on return.
- **Deploy behavior:** a production deploy restarts DOs — every socket drops once and reconnects into a fresh snapshot; state survives in DO SQLite. Reconnect logic is a **release-safety feature**; migrations expand-first so reconnecting clients from release N−1 tolerate release N schemas.
- **Per-DO soft throttle:** the DO sheds excess per-event load gracefully (calm retry states, never raw errors); DO soft limit ~1 000 req/s is far above the ~500-participant envelope.

## 10. Scheduling & purge architecture (decided)

- **The DO purges itself.** Each EventRoom multiplexes its **single alarm** (verified: exactly one per DO; official pattern) across a `schedule` table: announcement expiry → presence-grace expiries → auto-end broadcast (ends_at + 24 h) → **self-purge at end + 3 d** (`storage.deleteAll()` + alarm cleanup). Alarms wake idle/hibernated DOs, so purge fires without traffic. DO storage has **no time-travel** — content deletion is real.
- **Worker cron = D1-side enforcer + backstop** (same schedule both envs): expired `auth_tokens` cleanup · inactive-account purge (1 month; carve-outs: future event scheduled, M8 non-empty wallet) · `audit_log` > 12 months · KV `code:*` removal on event deletion · **defensive sweep**: any event whose purge is overdue in D1 terms gets its DO re-armed (lost-alarm insurance).
- **Every lifetime = one job + one deletion-proving test** (F010 rule; M12 verifies/monitors, never introduces).
- **M7 seams (reserved now, built then):** cron + DO alarm handlers check the emergency flag before purging (overdue purges run 24 h after resume); the emergency **flag itself is NOT the KV config mechanism** — M7 spec owns its stronger-semantics design (freeze is push-driven through DOs; the flag blocks new activity). Emergency exports: pre-generated static files in **R2**, download endpoint `/exports/⟨random token⟩` (token + ~24 h expiry in D1), **freeze-exempt, read-only, session-auth-free (the capability link is the auth), per-IP rate-limited**, R2 lifecycle auto-delete. Normal M5 exports are on-demand streams — nothing stored.

## 11. Auth & identity design (decided at interview)

### 11.1 Magic links (OQ7 — closed)
`POST /api/auth/request {email}` → Turnstile (from M4) → limits (§15) → **always 200, uniform timing** (send via `ctx.waitUntil`); banned/soft-deleted emails silently skip the send. Token: 32 random bytes → base64url in link; **SHA-256 hash stored in D1 `auth_tokens`** with `expires_at = now + 15 min`; **single-use enforced atomically** (`DELETE … RETURNING` on verify) — interview correction: D1, not KV, so replay is structurally impossible. Verify: expired/unknown ⇒ calm "request a new link" screen; success ⇒ user upserted (registration = first verify; M8 lockdown gates the *request* step), session issued. E-1 goes through the template registry (§14).

### 11.2 Moderator sessions
**Stateless** `ml_session` cookie: `uid.exp.HMAC-SHA256` (Web Crypto, `SESSION_SECRET`, constant-time compare), ~30 d, HttpOnly + Secure + SameSite=Lax. `requireUser()` = verify signature **and** load the D1 user row — ban/soft-delete/deletion bite on the next request despite statelessness. `requireAdmin()` = same + role check (role granted at login when email ∈ `ADMIN_EMAILS`). No server-side session store; no "log out everywhere" (only `SESSION_SECRET` rotation — documented operator lever).

### 11.3 Participant identity (FR-5 — closed)
- **Per-event, no global identifier:** signed cookie `ml_p_⟨eventId⟩` = `{pid: random 128-bit, ack?: true}` + HMAC; minted lazily on first join. Cross-event correlation structurally impossible.
- **Lifetime (operator-corrected):** cookie `Max-Age` = current `ends_at` **+ 48 h** (covers the ±24 h reschedule envelope + auto-end), refreshed on every HTTP response; the 3-day purge window is a server-side concept and does **not** govern the cookie. DO participant rows die with the content purge.
- **Leave event** = clear cookie + DO marks session left (identity reset per B6/T1-2; prior questions keep their attribution snapshot; Turnstile re-runs on rejoin once M4 ships).
- **Ack flag lives only in the cookie** — the acknowledgment gate leaves zero server-side trace; leave/rejoin re-prompts the primer.
- **Names/nicknames live in the DO only** (event content) — never in the cookie; no PII on the wire per request.
- **Display prefs = localStorage, explicit overrides only (precedence rule, operator-ratified):** localStorage stores *only explicit user adjustments*, keyed per event + surface; every render computes the server-derived default fresh (I18N-02 derivation: participant surface = device language; board = event language) and applies an override only if one exists for this event+surface. Server-side changes therefore propagate unless deliberately overridden; no event inherits another's override. Edge cases (register-variant, dual-surface viewer, pre-start event-language change) resolve in the F012/I18N-02 spec against this rule.

### 11.4 Admin wall (OQ1 — closed: **Cloudflare Access + one-time PIN**)
- Access application covers `/admin*` on **both hostnames**; IdP = one-time PIN; policy = operator email allowlist. Zero Trust free tier covers this (1 seat needed; free-plan seat count historically 50 — **[verify @M6]**).
- **Defense in depth (ASSUMED, ratified):** the Worker validates the Access JWT (`Cf-Access-Jwt-Assertion`) on every `/admin*` request and **fails closed** if absent/invalid — the wall is never dashboard-config-only. Inside, `requireAdmin()` still applies (ADM-01 unchanged).
- **Reset story (the hard requirement, by construction):** wall managed entirely in the Cloudflare dashboard — lockout is fixed there, zero deploys. Exit cost: delete the Access app, swap a TOTP wall behind the same `/admin` guard seam.
- Local dev: no Access locally — `ENVIRONMENT=dev`-guarded bypass + fake admin session in miniflare only (never in deployed envs — deployed dev is walled too).
- Step-up on destructive actions (M7 ceremony owned by M7 spec; direction decided): fresh Access re-authentication + the product's typed confirmation.

## 12. Board capability token (closed)
`crypto.randomUUID()` (operator choice: UUID form) minted at event creation (F001), stored unique+indexed on the event row; route `/b/⟨uuid⟩`; page + board-WS upgrade validate it. Never derived from event id or join code. **Regenerate (M5):** overwrite in one write — old URL dies instantly and the DO kicks connected board sockets. Anti-enumeration: 122 random bits + the same calm "unknown" screen as unknown codes + per-IP lookup rate limit. No KV mirror (board lookups are rare; one indexed D1 read).

## 13. Platform config, flags, log level (OQ8 + FND-06 — closed)
- **KV `CONFIG` namespace**: `platform:config` = ONE JSON document validated by a zod schema in `packages/shared` that also defines **every committed default** (empty KV ⇒ working platform; stored values override; malformed config ⇒ defaults + `critical` log, never crash). Flags = individual keys (`flag:event-creation-disabled`, `flag:maintenance-banner`, `flag:invite-only` @M8). `platform:log-level` = own key.
- Read via one `config.ts` service with **~30 s in-isolate cache**; worst-case propagation ≤ ~90 s (KV edge + cache) — operator-accepted for every value incl. log level. DOs read through the same binding + cache.
- Catalog (initial): create-ahead window (14 d default, FR-3) · reschedule window ±24 h · participant caps (100/1 000) · event limits (1+1) · question-length platform bounds 1–280 · announcement lifetime (~60 s direction) · reconnect grace (~60 s direction) · purge offsets (content 3 d, auto-end +24 h, accounts 1 mo, audit 12 mo, resume grace 24 h) · text caps (confirmation statement, hide-message @M5, decline reason @M9) · export-link validity ~24 h (M7) · curated submission-limit default (M9).
- Operator surface: M1 = documented `pnpm wrangler kv key put …` runbook commands; M6 admin UI writes the same keys through the same service. **The M7 emergency-stop flag is explicitly NOT this mechanism** (§10).

## 14. Email architecture (registry decided; findings re-verified 2026-07-14)
- **Verified current:** binding key `send_email` (name e.g. `EMAIL`); API `env.EMAIL.send({from, to, subject, text, html, …})` → `{messageId}` (legacy EmailMessage/mimetext path exists; not needed). Limits: 5 MiB/message, ≤50 recipients. **Pricing published — Workers Paid: 3 000 sends/month included, then $0.35 per 1 000; not available on Free plan.** New accounts start with conservative auto-scaling daily quotas (harmless at magic-link volume). Local dev **simulates** sends (content written to local files — that's where the magic link is read in dev); `remote: true` on the binding opts into real sends from local. Domain onboarding (row 2) auto-creates SPF/DKIM/DMARC + bounce-subdomain MX; one onboarding serves both envs. **[verify @F002]**
- **Template registry (T8b-5):** one module per email type in `worker/emails/` — E-1 magic link (M2) · E-2 feedback (M4) · E-3 force-end (M6) · E-4/E-4b/E-5 emergency set (M7) — each exporting `subject/text/html(locale, params)`, every string through Lingui, rendered in the **moderator's console language**. `worker/services/email.ts` is the **only** send path (Article-II seam; fallback candidates if ever needed: Resend/Postmark behind the same seam + decision-log entry) and accepts **only registered templates** — ad-hoc sends are impossible by construction. CI renders every registered template in every locale.
- **react.email: evaluated, not adopted** (interview decision): plain-text-first + one shared minimal HTML layout (inline styles, no images, no tracking) serves six short transactional mails; the registry's render seam keeps react.email a drop-in if M11 wants designed mail.
- Sender: `EMAIL_FROM` var, default `no-reply@micline.app` [ASSUMED]; display name at M2 spec.

## 15. Abuse & hardening stack
- **Turnstile (M4, SAFE-01):** invisible widget on exactly three actions — auth request, event creation, first submission per participant session (re-run after leave/rejoin). One widget, both hostnames (free plan: 20 widgets × 10 hostnames — fits). Server-side siteverify from the Worker; local dev + CI use Cloudflare's documented test keys (always-pass/always-fail/token-spent). **[verify @F006]**
- **Rate limits (SAFE-02 — keys + mechanisms decided; numbers at F006 spec, revisited at the OQ6 post-tech limit review):** one interface `worker/services/ratelimit.ts`; **Tier A** = Workers rate-limiting binding (verified: 10 s or 60 s windows only, per-colo, approximate/permissive — burst brake, never an accountant); **Tier B** = durable counters (D1 for account-scoped: magic-link per-email via `auth_tokens` count, event-creation per-uid daily, reschedule per-event daily, feedback per-uid daily; DO for event-scoped: per-sid submission bursts, per-event ceilings/soft throttle). Matrix: magic-link = per-email primary + generous per-IP burst · creation = per-uid · **reschedule = 60 s brake + per-event daily cap** (T8-5) · submission = per-sid + one-active-item (the real limiter) + per-event DO ceiling · join/board lookups + WS upgrades = per-IP with **room-scale ceilings** (venue-NAT rule §2.8; Cloudflare's DDoS layer fronts everything).
- **Headers & transport (F000 baseline, tightened F007):** HSTS (prod), nosniff, `frame-ancestors 'none'`, CSP report-only → enforced at F007; WS upgrades require an allow-listed Origin (micline.app, dev.micline.app, localhost) — checked before any auth work.
- **Content rules:** §8. **Threat map:** file 03 §G (one-screen table) stays authoritative for pipeline threats.

## 16. Observability & logging
Structured JSON logs (one logger, request-id + event-id correlation) → Workers Logs; global level from `platform:log-level` (debug ↔ critical, ≤ ~90 s propagation, FND-06 closed). No third-party sinks in current scope; the M12 OPS-01 decision picks the long-term pipeline against measured needs. Metrics visibility ladder (T9-4): participants none · moderators in-event count only · admin metrics wall (M6, own counters) · raw infra operator-only.

## 17. CI/CD & delivery
Five required checks (`lint` `typecheck` `test` `build` `secrets`) + CodeQL/Dependabot/push-protection as background gates; deploys ONLY via Actions (merge→DEV; published Release→PRD with tag checkout + human approval; prereleases allowed; patch releases defects-only). Workflow hardening: SHA-pinned actions, top-level `permissions: contents: read`, no untrusted-text interpolation, Environment-scoped secrets. Migrations: `wrangler d1 migrations apply --env ⟨e⟩ --remote` in the pipelines only; local = miniflare. **Expand-first schema rule** (§9 deploy behavior). Full lifecycle + release mechanics: file 03 §D/§F.

## 18. Test strategy
- **Unit (Vitest, workers pool):** every state-machine transition + invariant (F003 starts the **≥80 % service/DO coverage gate**), auth flows (token consume atomicity, cookie tampering, anti-enumeration timing), config-schema defaults/fallback, email templates × locales, rate-limit interface.
- **Integration:** two scripted WS clients through the full M3 script (create→open→join→submit "saved as #N"→second-submit blocked→skip+stub→undo→answered auto-pull→you're-up→withdraw→announcement expiry→reconnect-snapshot→end); presence/grace admission tests; **deletion-proving tests per lifetime** (F010).
- **E2E (Playwright, from M4):** the release-gate smoke scripts (file 06) automated where cheap; two-device flows stay manual smokes.
- Miniflare simulates D1/KV/DO/email locally; cron handlers testable via the local scheduled endpoint. Turnstile: test keys.

## 19. Cloudflare product facts (verified 2026-07-14 — re-verify per §22 at use time)
| Product | Facts that shaped decisions |
|---|---|
| Workers Paid | $5/mo; 10 M req + 30 M CPU-ms incl.; unlocks DO/D1/KV allowances; cron free (15-min CPU cap/invocation). Free plan: 100 k req/day (dev-only relevance) |
| Durable Objects | SQLite-backed on Free + Paid; 10 GB/object (Paid); ~1 000 req/s soft/DO; hibernation billing pause; auto ping/pong without wake; **exactly one alarm/DO**, at-least-once, ~6 retries w/ backoff, wakes hibernated DOs; in-memory state resets on hibernation |
| D1 | 10 GB/db (Paid); 1 000 queries/invocation; 100 bound params/query; **Time Travel always-on, non-disableable: 30 d Paid / 7 d Free** |
| KV | 25 MiB values; 1 write/s per key; eventually consistent (≤ ~60 s global); paid = effectively unmetered at our scale |
| Email Service | §14 — pricing published (3 000 incl. + $0.35/1k, Paid only); `send_email` binding confirmed; simulated local sends |
| Turnstile | Free: 20 widgets × 10 hostnames; unlimited (site)verify; documented test keys; invisible mode available |
| Rate-limit binding | 10/60 s periods only; per-colo; approximate/permissive (⇒ two-tier design §15) |
| Access (Zero Trust) | One-time PIN IdP; seat consumed per authenticated user; free tier sufficient for 1 admin (seat count **[verify @M6]**); JWT assertion validatable in-Worker |
| Wrangler | bindings/vars non-inheritable per env; env-level `name` override; per-env cron triggers; local scheduled-handler test route |

## 20. Decisions owned by later specs (recorded, not dropped)
| Decision | Owner |
|---|---|
| Exact numeric rate limits + per-DO throttle values | F006 spec (+ OQ6 limit revisit before the M4 beta gate) |
| Reconnect-grace final number (direction 60 s) | F003 spec |
| Sender display name; magic-link screen copy; anti-enumeration timing details | F002 / M2 spec |
| Nickname uniqueness per event + generation language (direction: event language); profanity block-vs-mask + wordlist source | F003/F005 specs (FR-5) |
| Failure-state copy (closed list), locale edges (register-variant, dual-surface) | F012 spec |
| PITR-honesty sentence wording (privacy page + deletion UI) | F007 / F011 specs |
| Access-wall integration details, force-end wording, consequence previews | M6 spec |
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
- **F000:** current `@cloudflare/vite-plugin` scaffold + official RR template (pick the RR major, record in §1) · Lingui × chosen-RR-major × Vite integration + catalog-in-build wiring · wrangler env config shape (non-inheritance, custom domains, per-env crons) · gitleaks/CodeQL/Dependabot setup currency · pnpm/Node current pins.
- **F008:** KV binding API + `wrangler kv key put` syntax; min `expirationTtl`/`cacheTtl` values.
- **F001:** D1 migrations commands per env · Drizzle/D1 driver currency · QR generation lib choice.
- **F002:** exact `send_email` wrangler shape + `send()` signature + local-dev file paths (fetch email-service/llms.txt again) · Email quotas for a new account · KV fine print if any KV use lands here.
- **F003:** DO hibernation API names + serializeAttachment size limit · alarm retry semantics · `durable-sqlite` Drizzle driver status · WS message-size limits · vitest-pool-workers DO testing patterns.
- **F006:** rate-limit binding config shape/status · Turnstile siteverify contract + current test keys.
- **F010:** cron trigger syntax + local testing route · D1 batch/param limits for purge jobs.
- **M6:** Zero Trust free-plan seat count + Access app/policy setup + JWT validation library/pattern (fetch cloudflare-one docs) — then log row 5.
- **M7:** R2 bucket + binding + lifecycle rules + multipart/streaming write patterns; D1 Time-Travel retention re-check for the honesty copy.
- **Ongoing:** any Cloudflare beta-status change for Email Service; Workers pricing table at each milestone kickoff (02-setup §2.9 ritual).
