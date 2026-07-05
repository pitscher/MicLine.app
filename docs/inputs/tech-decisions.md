# MicLine.app — Tech decisions (input to prompt P2)

> Provenance: distilled 2026-07-04 from an earlier deep planning round (Claude Opus 4.8) plus live-doc verification done the same day; toolchain + framework-major rows re-verified 2026-07-05. Status: **strong defaults, not gospel** — P2 must reconcile everything against current Cloudflare docs and against the product docs (which win on conflict). This file lives at `docs/inputs/tech-decisions.md`.
>
> **Lifecycle (kept deliberately, without conflicting with Spec Kit):** this file exists to feed prompt P2 *once*. P2 folds its content — corrected — into `docs/engineering/tech-context.md`, and **from Gate 2 on, only tech-context.md is authoritative**: plans, addenda, and CLAUDE.md cite it, never this file (see `docs/build-guide/04-product-definition.md`, Gate 2). This file then remains as provenance — the record of pre-Spec-Kit decisions. Do not update it after Gate 2; changes go to tech-context.md and the decision log.

## Product in one line
Event hosts collect, order, and moderate audience questions in real time — a shared "mic line": participants submit + upvote with no login; the moderator curates an ordered speaking queue (Now / Up next / line); each participant sees their live position and gets a "you're up" cue.

## Locked constraints
Public GitHub repo → zero secrets in the tree, ever. All compute/storage/delivery/email/bot-defense on Cloudflare; external APIs only where no Cloudflare primitive exists (future payments), behind a service seam. Solo operator: boring > clever, minimal dependencies, low standing cost. Free product in M1/M2 (monetization parked).

## Stack (pinned defaults)
| Concern | Choice |
|---|---|
| Runtime | Cloudflare Workers, single Worker, `nodejs_compat`, TypeScript strict (TS 6 current at verification — use current stable) |
| Web framework | React Router **framework mode** + React 19, SSR, built with Vite + `@cloudflare/vite-plugin`. v7 was the planning-time default; **v8 is current as of 2026-07** — P2/F000 pick the major against the current official Cloudflare template |
| API/WS routing | Hono mounted inside the same Worker for `/api/*` and `/ws/*`; everything else falls through to the RR request handler |
| Realtime | One Durable Object per event (`EventRoom`), WebSocket **Hibernation API**, DO SQLite storage |
| Global DB | D1 + Drizzle ORM + drizzle-kit migrations |
| Per-event DB | DO SQLite via Drizzle `durable-sqlite` driver |
| Cache/hot reads | Workers KV (event-code→id map; hashed magic-link tokens) |
| Email | Cloudflare Email Service binding (see verified findings below) |
| Bot defense | Turnstile (server-side siteverify) |
| Rate limiting | Workers rate-limiting binding if available on plan; else KV-counter fallback behind the same interface |
| Monorepo | pnpm 11 workspaces + Turborepo (exact pnpm pinned via `packageManager`, Node 24 LTS via `.node-version` — docs/build-guide/02-setup.md §2.1): `apps/web`, `packages/shared` (zod schemas, WS contracts, Drizzle schemas), `packages/ui` (design tokens) |
| Quality | Biome (lint+format), Vitest + `@cloudflare/vitest-pool-workers`, Playwright later |
| Styling | Tailwind CSS v4, tokens centralized in `packages/ui` |
| IDs/codes | `crypto.randomUUID()` for ids; event join codes = 6-char Crockford base32 (no I/L/O/U), case-insensitive input, collision-checked with retry; join URL `/e/⟨CODE⟩` |
| i18n | Lingui (@lingui/react + macros, PO catalogs), locales `en`+`de` at launch, extraction + catalog compile wired into `build` so unextracted strings fail CI — verify current React Router + Vite integration at F000 |

## Architecture rules
1. **Service layer is the API.** All business logic lives in `worker/services/*` as plain functions `(env, …)`. Hono routes AND React Router loaders/actions call these directly — loaders never HTTP back into the same Worker.
2. **The DO is the single realtime authority.** It owns live state, serializes all mutations (single-threaded actor), broadcasts snapshot-then-deltas, and enforces the invariants below. Clients render server state; they never compute authoritative state. Edge (Worker) authenticates and passes role via headers/socket tags; the DO trusts tags but re-checks role on mutating messages.
3. **Auth = magic links only.** Email is an identifier, never a credential: 32-byte random token → store only its SHA-256 hash in KV (TTL ~15 min, single-use) → signed HttpOnly session cookie (HMAC-SHA256 via Web Crypto, `SESSION_SECRET`, constant-time compare, ~30-day exp; cookie flags HttpOnly + Secure + SameSite=Lax). Participants get an anonymous signed `sid` cookie (no PII) driving vote-dedupe and authorship. Anti-enumeration: the request endpoint always returns 200.
4. **Question-line domain (the heart).** Statuses: `pending → pool → queued → speaking → answered/dismissed` (pre-moderation mode inserts at `pending`; open mode at `pool`). Invariants: at most one `speaking` per event; `linePosition` (fractional index for O(1) reorder, periodically renormalized) is non-null **iff** `queued`; one upvote per (question, sid) enforced by PK; no self-upvote; terminal states immutable. "Up next" = smallest `linePosition` (derived). Visibility: participants never see others' `pending`/`dismissed` nor any `authorSessionId`.
5. **Entitlement seam (constitution article).** Anything gated is checked only through one `can(actor, capability, ctx)` service; grants come from a baseline + vouchers (M2), later possibly billing — gated code never knows the source.
6. **Forward-compat pulled into M1:** `users.role` ('user'|'admin') + `requireAdmin()`; append-only audit-log table; soft-delete semantics; structured logs; design tokens only (no one-off styling); a decided data-retention policy for ended events (enforcement may ship M5).

## Abuse, content & hardening defaults
Turnstile on: auth request, event create, first submission per participant session. Rate limits (per action, keyed ip/uid/sid). Question text: trim, 1–280 chars, collapse whitespace, strip control chars; optional wordlist profanity filter (default off). Security-header baseline from F000 (HSTS in prod, nosniff, `frame-ancestors 'none'`, CSP report-only → enforced by F007); WebSocket upgrades require an allow-listed Origin. Full threat map: `docs/build-guide/03-github-operations.md` §G.

## Environments & delivery
Wrangler environments `dev` (worker `micline-dev`, dev.micline.app, own D1/KV) and `production` (worker `micline`, micline.app, own D1/KV). Deploys only via GitHub Actions: merge→DEV, published Release→PRD (human-approved). Secrets: `.dev.vars` locally, `wrangler secret put --env ⟨e⟩` remotely, GitHub environment secrets for CI. Full design: `docs/build-guide/03-github-operations.md`.

## Verified findings — Cloudflare Email Service (as of 2026-07-04; re-verify in F002)
Configured as a **`send_email` binding** in the Wrangler config; Workers API: `env.EMAIL.send({ to: [{email}], from: {email, name}, subject, text|html })`. Sending domain must be onboarded once (Dashboard → Compute → Email Service → Email Sending) — Cloudflare auto-creates SPF/DKIM/DMARC (+ MX on a `cf-bounce` subdomain). Local `wrangler dev` **simulates** sends by default (logged + written to local files — that's where you read the magic link in dev); set `remote: true` on the binding to send real mail from local dev. Max message 5 MiB. Requires Workers Paid; product is beta and per-message pricing is not final ⇒ **email stays behind `worker/services/email.ts` as a swappable seam** (fallback candidates: Resend/Postmark via REST — a permitted Article-II exception if ever needed).

## Verify-at-implementation list
Lingui × React Router (the major chosen at F000) + Vite integration and catalog build wiring (F000) · Exact `send_email` wrangler key/shape + `send()` signature (F002, from developers.cloudflare.com/email-service/llms.txt) · rate-limiting binding availability on Workers Paid (F002) · current `@cloudflare/vite-plugin` scaffold incl. the React Router major to build on — v8 is current, v7 was the planning default (F000) · DO Hibernation + SQLite APIs and `durable-sqlite` driver status (F003) · Turnstile siteverify contract + test keys (F006) · D1 migration commands per environment (F000/F001).
