# MicLine.app — Feature Inventory

> Provenance: P1 interview 2026-07-06 · **P2 final product interview 2026-07-12** · **P3 milestone restructuring and release versioning 2026-07-12**. Status values: **confirmed** (explicit interview decision) · **assumed** (Claude's call under interview rules, veto anytime) · **open** (deferred, owner noted) · **parked** (deliberately shelved) · **rejected** (decided against; kept for context). Current delivery order: **M1 → M2 → M3 → M4 → M5 → M6 → M7 → M8 → M9 → M10 → M11 → M12 → M13-if-ever**. Stable feature IDs and all P1/P2 scope are preserved; P3 changes sequencing, milestone ownership, and target versions only.

## Release-version policy

Each completed milestone is a releasable MicLine version. Development builds may use SemVer prerelease suffixes such as `-alpha.N`, `-beta.N`, and `-rc.N`; patch releases fix defects without adding milestone scope. Parked and rejected ideas have no target version. M13's `v1.1.0` is provisional and may move if other post-`v1.0.0` releases intervene.

| Milestone | Name | Target version | Release outcome |
|---|---|---:|---|
| M1 | Deployable foundation | `v0.1.0` | A secure, continuously deployable MicLine application shell exists in development and production. Core architecture, delivery, configuration, logging, localization plumbing, security boundaries, and foundational test gates are established; there is no usable event product yet. |
| M2 | Moderator identity and event setup | `v0.2.0` | A moderator can authenticate, maintain the deliberately minimal account, create and schedule an event, and obtain its join code, URL, QR code, and board capability URL. |
| M3 | Open-line event runtime | `v0.3.0` | MicLine can run a complete internal FIFO event: participants join without a gate, submit and withdraw questions, follow the line and position cues, receive announcements, and use the projector board; the moderator operates the line. |
| M4 | Public-beta readiness | `v0.4.0` | The internal alpha becomes responsible to expose to real users. Retention and deletion are enforced, abuse controls and failure states are complete, English and German are production-ready, and the public/legal and operational baseline exists. |
| M5 | Moderator control and data portability | `v0.5.0` | Moderators gain live-event safety controls, board-link recovery, follow-up flags, richer recap, and complete exports. |
| M6 | Operator control plane | `v0.6.0` | The protected admin interface supports normal oversight, abuse handling, force-end, deletion, metrics, statistics, service notices, platform controls, feedback configuration, and consequence previews. |
| M7 | Emergency operations and recovery | `v0.7.0` | MicLine gains total emergency-stop, emergency export and communication, job suspension and controlled resumption, purge grace, emergency screens, and the separate delete-all-data procedure. |
| M8 | Entitlements, invite codes and lockdown | `v0.8.0` | The entitlement engine, vouchers, wallet, capability and limit grants, revocation, transparency, and invite-only registration/event creation become operational. |
| M9 | Curated moderation | `v0.9.0` | MicLine gains the curated pool and line, upvotes, review and decline flows, duplicate merging, submission limits, console efficiency tools, per-viewer pool controls, and co-moderation. |
| M10 | Extended event toolkit | `v0.10.0` | Lower-priority event extensions ship on the stable moderation model: timer, duplicate/templates, markdown-lite primer and announcements, in-event primer access, vanity slugs, formal German, and the mature translation-management pipeline. |
| M11 | Design system, identity and experience overhaul | `v0.11.0` | The visual system and identity are designed and applied across every surface, including board appearance, optional sound cue, and event branding. |
| M12 | Operational maturity and compliance | `v1.0.0` | Measured observability, cost modelling, budget alerts, load testing, operational audits, recovery runbooks, retention verification, and the compliance pack establish the first production-mature release. |
| M13 | Monetization, if ever | *provisional `v1.1.0`* | The deliberately parked commercial layer: billing as another entitlement grantor, compensation policy, payment communications, and the German legal and tax package. |

### Release gates

- **After M3 / `v0.3.0`:** internal end-to-end alpha.
- **After M4 / `v0.4.0`:** first responsible public beta.
- **After M8 / `v0.8.0`:** operationally controlled public service.
- **After M9 / `v0.9.0`:** differentiated MicLine product.
- **After M12 / `v1.0.0`:** first production-mature release.

### Legacy build-plan cross-references

References to F000–F007 and `docs/build-guide/06-m1-plan.md` are retained as provenance. That legacy plan described the former single-M1 implementation bundle, which is now distributed across M1–M4. Until regenerated or reconciled after the tech interview, it must not be treated as the current milestone authority; this inventory controls product scope and sequence.

## M1 — Deployable foundation

**Target release:** `v0.1.0`

A secure, continuously deployable MicLine application shell exists in development and production. Core architecture, delivery, configuration, logging, localization plumbing, security boundaries, and foundational test gates are established; there is no usable event product yet.

| id | name | description | milestone | target version | status |
|---|---|---|---|---:|---|
| FND-01 | Monorepo & delivery pipeline | pnpm/turbo workspace, CI (lint/typecheck/test/build/secrets), DEV/PRD deploy via GitHub Actions only (F000). Tech interview must first outline the complete Cloudflare + GitHub setup; foundation is revisited once the bare-minimum end-to-end alpha exists at M3 / `v0.3.0`. | M1 | `v0.1.0` | confirmed |
| FND-02 | i18n plumbing | Lingui with `en` + `de`, every user-facing string extracted, unextracted strings fail CI (F000). Locale derivation model: see I18N-02. | M1 | `v0.1.0` | confirmed |
| FND-03 | Security & hardening baseline | Security headers, WS origin allowlist, strict TS/zod boundaries, structured logs, audit-log table (privileged admin/entitlement actions only — never moderator in-event actions), soft-delete semantics, `users.role` column. | M1 | `v0.1.0` | confirmed |
| FND-04 | Platform config & kill-switch flags | Central code-change-free config (retention windows, limits, P2 values: announcement lifetime, text caps, link validity, purge grace, curated participant-limit default) + KV flags: event-creation kill-switch, maintenance banner — wrangler CLI in M1 / `v0.1.0`; admin UI wrapper in M6 / `v0.6.0`; invite-only control activates with M8 / `v0.8.0`. | M1 | `v0.1.0` | confirmed |
| FND-06 | Global log-level control | Deployment-wide log level (debug ↔ critical), freely adjustable by the operator without code change; details at tech interview. | M1 | `v0.1.0` | confirmed |

## M2 — Moderator identity and event setup

**Target release:** `v0.2.0`

A moderator can authenticate, maintain the deliberately minimal account, create and schedule an event, and obtain its join code, URL, QR code, and board capability URL.

| id | name | description | milestone | target version | status |
|---|---|---|---|---:|---|
| AUTH-01 | Magic-link auth | Email-only moderator auth (E-1): hashed single-use token (15 min), ~30-day session cookie, anti-enumeration; banned addresses silently stop receiving links (F002). | M2 | `v0.2.0` | confirmed |
| AUTH-02 | Durable-minimal accounts | Stored: email, optional display name, country (CF geolocation, aggregates only), persisted console language; nothing else. | M2 | `v0.2.0` | confirmed |
| EVT-01 | Event creation | Create up to 48 h ahead (configurable); required: title, start & end date+time; optional: private description, primer (+ acknowledgment mode + custom confirmation statement); settings: language, name mode, profanity filter, per-event min/max question length within platform 1–280 (F001). | M2 | `v0.2.0` | confirmed |
| EVT-02 | Event scheduling lifecycle | Scheduled → auto-opens at start → ended manually or auto-end at end_time+24 h; start/end each adjustable ±24 h around original values, unlimited count within window (rate-limited action); bigger moves = delete + recreate (new join code). | M2 | `v0.2.0` | confirmed |
| EVT-03 | Settings freeze | Event settings immutable once live (scheduling fields keep their ±24 h window; live controls — announcements, and from M5 / `v0.5.0` intake close / hide board / hide-message — are not settings). | M2 | `v0.2.0` | confirmed |
| EVT-04 | Join codes & share panel | 6-char Crockford code, `/e/⟨CODE⟩` URL, QR; share panel in console (F001). | M2 | `v0.2.0` | confirmed |
| MOD-03 | My-events list | Dashboard: scheduled / active / past events, sorted by date. M4 / `v0.4.0` adds past-until-purge and deletion-countdown states; M6 / `v0.6.0` adds the "removed by administration" state. | M2 | `v0.2.0` | confirmed |

## M3 — Open-line event runtime

**Target release:** `v0.3.0`

MicLine can run a complete internal FIFO event: participants join without a gate, submit and withdraw questions, follow the line and position cues, receive announcements, and use the projector board; the moderator operates the line.

| id | name | description | milestone | target version | status |
|---|---|---|---|---:|---|
| LINE-01 | Open line (FIFO) | Submitted question appended straight to the public line; order = arrival; the sole runtime mode in M3 / `v0.3.0`; curated mode arrives in M9 / `v0.9.0`. Position stability ("your position only ever improves") is an open-line guarantee. | M3 | `v0.3.0` | confirmed |
| LINE-02 | Full-line transparency | All participants see the whole ordered line + Now speaking; positions visible to everyone. | M3 | `v0.3.0` | confirmed |
| LINE-03 | Moderator ground control | Mark top question answered (next auto-pulled); skip any item — skip removes the text from all public surfaces, a stub (attribution + "skipped") remains in the participant line, the board never shows skipped items, the owner keeps a "skipped" chip. | M3 | `v0.3.0` | confirmed |
| LINE-04 | Undo | ~10 s single-level undo after answered and skip. | M3 | `v0.3.0` | confirmed |
| LINE-05 | Withdraw own question | Participant removes their own not-yet-called question; slot frees instantly. | M3 | `v0.3.0` | confirmed |
| LINE-06 | Position banner & cues | Sticky "You're #N" → "You're up soon" (approaching the top) → full-screen "You're up!" on the participant surface. | M3 | `v0.3.0` | confirmed |
| LINE-07 | Realtime engine | One Durable Object per event, WebSocket hibernation, snapshot+deltas, state-machine invariants, graceful reconnect (F003). Binding acceptance criteria: no silent loss · duplicate-free retries · explicit save states · automatic recovery (no manual refresh as designed path). | M3 | `v0.3.0` | confirmed |
| LINE-08 | One active question per participant | Open line: exactly one active waiting item per participant (fixed platform rule); answered/skipped/withdrawn frees the slot. Enforcement session-scoped; leave-and-rejoin residual accepted as best-effort (fingerprinting/IP binding rejected); copy claims "enforced per session". | M3 | `v0.3.0` | confirmed |
| PRT-01 | Zero-gate join | QR/URL/code → live board instantly; no account/name/cookie-banner/CAPTCHA/permission prompts on entry; anonymous session cookie. | M3 | `v0.3.0` | confirmed |
| PRT-02 | Join methods | QR + short URL + landing-page code box, all simultaneously valid. | M3 | `v0.3.0` | confirmed |
| PRT-03 | Name modes | Per-event: Nicknames (automatic, localized funny pseudonyms, default) or Real names (required before first submission, locked for the event, unverified). | M3 | `v0.3.0` | confirmed |
| PRT-04 | Name confirm & leave/rejoin | Explicit "you'll appear as ⟨Name⟩" confirm; Leave-event resets session (questions keep old attribution; Turnstile re-runs). | M3 | `v0.3.0` | confirmed |
| PRT-05 | Pre-join primer & acknowledgment | Optional plain-text briefing before joining; per-event choice: skippable vs required explicit confirmation with moderator-customizable statement (plain text, capped; localized default "I understand."). Acknowledgment is a gate, not a record — nothing stored per participant, not legal proof. | M3 | `v0.3.0` | confirmed |
| PRT-06 | Join-state screens | Lobby ("starts at ⟨time⟩"), event-full (no waitlist), ended, unknown/expired code. | M3 | `v0.3.0` | confirmed |
| PRT-07 | Own-question status | Chips: in line #N / answered / skipped (M9 / `v0.9.0` adds: in pool / declined / merged); withdrawn disappears. | M3 | `v0.3.0` | confirmed |
| BRD-01 | Projector board (minimal) | Read-only big-screen route behind its own unguessable capability URL (never exposes event ID or join code): QR + code corner, Now/Up-next huge, top of line, announcement area, fullscreen toggle; one layout, no settings; never shows skipped items, timestamps, or vote counts. | M3 | `v0.3.0` | confirmed |
| MOD-01 | Moderator console | One phone-capable screen: header (title, share panel, live participant count, end button), the line with actions, announcement composer (F004). | M3 | `v0.3.0` | confirmed |
| MOD-02 | Announcements | Live control: one current announcement, replaces previous, shown on board + participant surfaces; bounded display lifetime (platform value, ~60 s direction) with uniform auto-expiry everywhere; subtle draining progress cue on the board (style assumed); clear-early action; re-send restarts; no history/scheduling. | M3 | `v0.3.0` | confirmed |
| SAFE-03 | Content rules | Trim, platform 1–280 chars (per-event min/max within bounds via EVT-01), collapse whitespace, strip control chars. All moderator-authored text renders inert (no clickable/auto-linked URLs, no images). | M3 | `v0.3.0` | confirmed |
| SAFE-04 | Profanity filter | Optional per-event wordlist filter, default off (F006). | M3 | `v0.3.0` | confirmed |

## M4 — Public-beta readiness

**Target release:** `v0.4.0`

The internal alpha becomes responsible to expose to real users. Retention and deletion are enforced, abuse controls and failure states are complete, English and German are production-ready, and the public/legal and operational baseline exists.

| id | name | description | milestone | target version | status |
|---|---|---|---|---:|---|
| FND-05 | Translation status tracking | Lunaria dashboard/PR-comments loop (EmDash pattern), non-gating tooling loop after F001. | M4 | `v0.4.0` | confirmed |
| EVT-05 | End-of-event recap (on-screen) | Moderator sees stats recap after end with explicit content-deletion countdown; stats (survive) visually separated from content (dies); participants get polite end screen with "join another event" path. | M4 | `v0.4.0` | confirmed |
| EVT-06 | Event stats aggregates | Anonymous aggregates (counts, duration, country) snapshot at end; survive all deletions unlinkably. | M4 | `v0.4.0` | confirmed |
| EVT-07 | 3-day content retention | Event content purged 3 days after end; in-product countdown, no reminder email; purge jobs pause during an emergency stop, overdue purges run 24 h after resume (export ships M5 / `v0.5.0`; emergency pause/resume behavior activates in M7 / `v0.7.0`). | M4 | `v0.4.0` | confirmed |
| AUTH-03 | Self-service account deletion | "Delete my account and data" from the first public release, M4 / `v0.4.0` (GDPR Art. 17). Running event must be ended first; scheduled events listed in quantified consequence preview; UI is self-sufficiently final ("all your data has been purged"), no confirmation email. | M4 | `v0.4.0` | confirmed |
| AUTH-04 | Inactivity auto-purge | Account purged after 1 month inactivity, no warning email; carve-outs: future event scheduled, plus a non-empty wallet once M8 / `v0.8.0` ships; re-registration hint on auth screen. | M4 | `v0.4.0` | confirmed |
| PRT-08 | Failure states & state-takeover | Canonical closed list: delayed save, failed save + retry, "reconnecting — the line continues" banner, stale-board chip, event full, unknown code. Races resolve into the new canonical state with calm feedback, never raw errors. Console set: reconnecting, action-failed-retry, admin-ended notice, emergency notice. Tone: calm, honest, no error codes on participant surfaces. Device-agnostic ("participant surface", not phone). | M4 | `v0.4.0` | confirmed |
| MOD-04 | Feedback funnel (minimal) | Optional post-event feedback box; delivered by email (E-2) to FEEDBACK_EMAIL (Wrangler secret in M4 / `v0.4.0`; admin-UI setting in M6 / `v0.6.0`). | M4 | `v0.4.0` | confirmed |
| SAFE-01 | Bot defense | Invisible Turnstile on: auth request, event creation, first submission per participant session. | M4 | `v0.4.0` | confirmed |
| SAFE-02 | Rate limiting | Per-action limits keyed ip/uid/sid (Workers binding or KV fallback); rescheduling is a rate-limited action. | M4 | `v0.4.0` | confirmed |
| MKT-01 | Landing page | Moderator-pitch hero (line pillar), 3-step how-it-works, create CTA, prominent join-code box, honest privacy block, visual line mock; clean — depth in sub-pages; no pricing, no repo link, no SLA disclaimers on the hero. | M4 | `v0.4.0` | confirmed |
| MKT-02 | Sub-pages | How it works, FAQ (incl. honest outage entry), privacy (truthful lifetimes/deletion/export responsibility), terms (best-effort/no-SLA phrased as what MicLine does), Impressum, contact (F007). | M4 | `v0.4.0` | confirmed |
| I18N-02 | Surface locale derivation | Console: device default, adjustable anytime, persisted on account · Event language: device default at creation, adjustable before start only · Participant surface: participant device default, adjustable anytime, persisted in session/device · Board: event-language default, per-viewer adjustable, persisted per device until event end · Emails: moderator's console language. UI chrome only, never content. Spec edges: register-variant resolution, dual-surface viewer. | M4 | `v0.4.0` | confirmed |
| OPS-03 | Retention enforcement | Automated purge jobs for every lifetime in the data-lifecycle table (DO cleanup via alarms; emergency-stop pause + 24 h resume grace). | M4 | `v0.4.0` | confirmed |

## M5 — Moderator control and data portability

**Target release:** `v0.5.0`

Moderators gain live-event safety controls, board-link recovery, follow-up flags, richer recap, and complete exports.

| id | name | description | milestone | target version | status |
|---|---|---|---|---:|---|
| STD-05 | Recap & export | Rank 1 (raised from 5; closes OQ5): post-end recap + one-click export in CSV, JSON, and Line snapshot (HTML — self-contained one-pager mirroring the queue incl. pool; naming WIP). Contents: everything (questions, attribution, times, final statuses incl. declined + reasons, merge structure, votes, follow-up flags, primer, stats). No export-time filters. Responsibility line at export + privacy page. Entitlement-independent, never gated. Format list re-validated at M5 kickoff. | M5 | `v0.5.0` | confirmed |
| MOD-05 | Intake close | Moderator live control, manual toggle only (no timers/auto-close), freely reversible, works in both line modes (stops new submissions/pool intake); line stays visible and withdrawable; honest "the line is closed for new questions" copy; display details at spec. | M5 | `v0.5.0` | confirmed |
| MOD-06 | Hide board | Moderator live control: projector board shows event title + neutral message (system default, moderator-customizable at hide-time, platform cap) + wordmark; no line content, no QR/code while hidden. Board-only — participant surfaces and intake unaffected. No audit-log entry (ops logs only). Distinct from admin emergency stop. | M5 | `v0.5.0` | confirmed |
| MOD-07 | Regenerate board link | Board route uses its own unguessable capability token (never event ID/join code; no enumeration — tech requirement); moderator can invalidate and re-mint the board URL anytime (leak recovery). Adopted instead of the rejected board password. | M5 | `v0.5.0` | confirmed |
| MOD-08 | Follow-up flag | Moderator-only boolean on any item (pool/line/answered/skipped), set during event or recap window; never visible to participants, never affects public state; export column; dies with purge. | M5 | `v0.5.0` | confirmed |

## M6 — Operator control plane

**Target release:** `v0.6.0`

The protected admin interface supports normal oversight, abuse handling, force-end, deletion, metrics, statistics, service notices, platform controls, feedback configuration, and consequence previews.

| id | name | description | milestone | target version | status |
|---|---|---|---|---:|---|
| ADM-01 | Admin bootstrap | Admin role exclusively via ADMIN_EMAILS Wrangler secret (Cloudflare-side); no in-product admin management; recovery = update secret. | M6 | `v0.6.0` | confirmed |
| ADM-02 | Admin auth wall | /admin behind one dedicated mandatory auth layer (not magic-link + stacked factor). Mechanism (Cloudflare Access vs TOTP vs passkey) decided at the tech interview for the M6 specification with pricing/limits/exit-cost criteria; hard requirement: emergency reset always possible via Cloudflare account access alone — no lockout dead-end. Step-up confirmation on destructive actions regardless. | M6 | `v0.6.0` | confirmed (mechanism: open → tech interview) |
| ADM-03 | Oversight & abuse tools | Search events/moderators, detail views, read-only god view of live events incl. live states (board hidden, intake closed), force-end (calm outside: participants/board see normal ended state; truthful inside: moderator sees "ended by platform administration" + contact path + optional admin message + email E-3), soft-delete (owner sees "removed by administration"), ban moderator email (silent), GDPR full delete. | M6 | `v0.6.0` | confirmed |
| ADM-04 | Abuse doctrine | Participants = moderator's problem (skip-stub, hide board, primer, filters, rate limits); moderators/events = admin's; reports via contact email, no in-product report button. | M6 | `v0.6.0` | confirmed |
| ADM-05 | Metrics wall | Own-counter stats: events (created/active/week), participants, questions, emails sent, Turnstile failures, rate-limit hits, storage growth; € projection arrives with the M12 / `v1.0.0` cost model. | M6 | `v0.6.0` | confirmed |
| ADM-06 | Global stats view | Totals + anonymous aggregates: events hosted/planned, participants approx., moderator countries, event durations. | M6 | `v0.6.0` | confirmed |
| ADM-07 | Platform controls | Service notice banner (placement: always landing/auth/console; participant surfaces + board only when flagged as affecting live events), creation kill-switch (UI over the M1 / `v0.1.0` flags); invite-only mode toggle activates with M8 / `v0.8.0`. | M6 | `v0.6.0` | confirmed |
| ADM-10 | Feedback email setting | Admin-configurable dedicated feedback address (takes over from the M4 / `v0.4.0` secret). | M6 | `v0.6.0` | confirmed |
| ADM-11 | Consequence previews & reversal matrix | Every admin control shows a quantified consequence preview ("X events affected: X running, X scheduled, X in export window") with computation timestamp ("computed at ⟨time⟩ — may already be outdated") and defines its reverse action in the UI; nothing requires a redeploy to undo; irreversible parts labeled honestly. | M6 | `v0.6.0` | confirmed |

## M7 — Emergency operations and recovery

**Target release:** `v0.7.0`

MicLine gains total emergency-stop, emergency export and communication, job suspension and controlled resumption, purge grace, emergency screens, and the separate delete-all-data procedure.

| id | name | description | milestone | target version | status |
|---|---|---|---|---:|---|
| ADM-08 | Emergency stop | Total stop with typed confirmation + step-up: ends running events, blocks creation and registration, freezes all surfaces, moderator auth, and scheduled jobs incl. purges. Exactly three exceptions: static notice pages, admin UI (reversal), expiring export-download endpoint. Sequence: freeze → snapshot exports for affected events → emails E-4/E-4b (trigger-modal export choice, default ON, "skip if you suspect data compromise"). Scheduled events never open during a stop; on reversal open late if window current, else auto-end; overdue purges 24 h after resume; E-5 states deadlines. Participants/board: calm emergency-ended screen. | M7 | `v0.7.0` | confirmed |
| ADM-09 | Delete all data | Separate GDPR/breach nuke: step-up auth, typed confirmation, consequence preview, honest irreversibility labeling; D1 time-travel ≤30 days is an infra safety net, DO storage is not; operationally preceded by emergency stop (runbook). | M7 | `v0.7.0` | confirmed |

## M8 — Entitlements, invite codes and lockdown

**Target release:** `v0.8.0`

The entitlement engine, vouchers, wallet, capability and limit grants, revocation, transparency, and invite-only registration/event creation become operational.

| id | name | description | milestone | target version | status |
|---|---|---|---|---:|---|
| ENT-01 | Entitlement engine | Single `can(actor, capability, ctx)` seam; grants table with source (baseline/invite-code/admin/billing-future), scope, expiry, revocation; audit-logged. | M8 | `v0.8.0` | confirmed |
| ENT-02 | Free baseline | All features free (lockdown off); limits 100 participants/event, 1 active + 1 planned event. Never-gated: core Q&A mechanics + all P2 moderator tools (intake close, hide board, regenerate board link, pool settings/limits, decline reasons, follow-up flag, filters/search/bulk, recap/export). Gateable: co-mods, branding, vanity slugs, limit raises, registration. | M8 | `v0.8.0` | confirmed |
| ENT-03 | Invite codes (vouchers) | Bundles {may_register? + grants + limit raises + validity} redeemed into account wallet; multi-use via max_redemptions; two clocks (redeem-by, grant validity); god-voucher preset; design details (format, QR, custom message, generation UX, clock defaults) at M8 kickoff. | M8 | `v0.8.0` | confirmed |
| ENT-04 | Revocation semantics | Kill unredeemed code or revoke redeemed grant; effect at next gated act; never touches running events; rescheduling committed events is never a gated act. | M8 | `v0.8.0` | confirmed |
| ENT-05 | Invite-only mode (lockdown) | Gates new registration and new event creation (incl. duplicate-event) behind invite codes; existing events run/start normally; doubles as cost brake. | M8 | `v0.8.0` | confirmed |
| ENT-06 | Entitlement transparency | Account page shows active grants, meters remaining, validity. | M8 | `v0.8.0` | confirmed |

## M9 — Curated moderation

**Target release:** `v0.9.0`

MicLine gains the curated pool and line, upvotes, review and decline flows, duplicate merging, submission limits, console efficiency tools, per-viewer pool controls, and co-moderation.

| id | name | description | milestone | target version | status |
|---|---|---|---|---:|---|
| STD-01 | Curated line mode | Pool → moderator selects/orders/drag-reorders into the line; return-to-pool; skip-stub everywhere; duplicate merge (pool-only: moderator picks primary, ×N badge travels with it, merged items inherit primary's outcome, votes combine deduped, unmerge until consumed, primary's submitter gets the mic). Pool visibility per-event setting (hidden auto-disables upvotes); owner always sees own chip. Per-event max open submissions per participant (pool+line; terminal states free the slot). Defaults (pool visible, upvotes on) assumed — finalize at M9 kickoff. | M9 | `v0.9.0` | confirmed |
| STD-02 | Upvotes + pool sort | Participants upvote pool questions (nameless, session-deduped best-effort); participant pool sort votes-desc (or newest when off); votes advise curation, never reorder the line; no vote UI/counts on line items on participant surfaces or board; requires visible pool. | M9 | `v0.9.0` | confirmed |
| STD-03 | Review mode (pre-moderation) | Submissions need approval before becoming public, in either mode; open+review appends at approval time (approval order = line order); curated+review releases into pool; declined = owner-visible chip + optional short moderator reason (plain text, capped, inert), reversible until event end, never public, retained until purge; consequence hints at creation; no co-mod dependency. | M9 | `v0.9.0` | confirmed |
| STD-04 | Co-moderators | Invite helpers to review/curate simultaneously (event-scoped role); gateable; invitations link-based (no platform email) — mechanism at M9 kickoff. | M9 | `v0.9.0` | confirmed |
| MOD-09 | Console filters, search & bulk verbs | Minimal set: pool sort (votes/newest/oldest), review-tray views (pending/approved/declined), flagged-only view, per-submitter counts + submission timestamps (console-only), text search over pool/line/declined, multi-select with exactly two bulk verbs (merge, bulk decline — no bulk-approve). Full cross-surface filter matrix (incl. filter-by-submitter, participant/board display filters) at M9 kickoff with transparency guardrail: full line is always the default view; the board keeps one canonical layout. | M9 | `v0.9.0` | confirmed |
| PRT-09 | Per-viewer pool collapse | Participants and board viewers can collapse/expand the pool zone per device (persisted until event end); the line itself is never hideable; defaults (direction: pool collapsed on the board) at M9 kickoff. | M9 | `v0.9.0` | confirmed |

## M10 — Extended event toolkit

**Target release:** `v0.10.0`

Lower-priority event extensions ship on the stable moderation model: timer, duplicate/templates, markdown-lite primer and announcements, in-event primer access, vanity slugs, formal German, and the mature translation-management pipeline.

| id | name | description | milestone | target version | status |
|---|---|---|---|---:|---|
| STD-06 | Speaker timer | Elapsed time on Now card, optional soft countdown on the board. | M10 | `v0.10.0` | confirmed |
| STD-07 | Duplicate event → templates | One-click re-create with same settings (creation act — gated under lockdown); saved presets later. | M10 | `v0.10.0` | confirmed |
| STD-08 | Rich announcements & primer | Markdown-lite (structure only — subset at M10 kickoff; no links rendered clickable, no images) for primer + announcements; primer becomes re-openable in-event (ⓘ sheet on participant surface) — one container, no separate info panel. | M10 | `v0.10.0` | confirmed |
| STD-09 | Vanity join slugs | Custom slug alongside the code; gateable perk. | M10 | `v0.10.0` | confirmed |
| STD-11 | Rented TMS pipeline | Parallel track: Weblate/Crowdin (free OSS tier) syncing catalogs via PRs; replaces-or-feeds Lunaria — decided at M10 kickoff. | M10 | `v0.10.0` | confirmed |
| I18N-01 | de (formal) event language | Second German catalog (Sie) selectable as event language; participant surfaces + board only, moderator UI stays Du; register-variant edge (device `de` + event `de (formal)` → Sie direction) resolved at spec time. | M10 | `v0.10.0` | assumed |

## M11 — Design system, identity and experience overhaul

**Target release:** `v0.11.0`

The visual system and identity are designed and applied across every surface, including board appearance, optional sound cue, and event branding.

| id | name | description | milestone | target version | status |
|---|---|---|---|---:|---|
| DSN-01 | Design brief & system | Design interview building on captured direction (Luma north star, adjectives, dark-first, motion-explains-state, invisible host); tokens/components in packages/ui. | M11 | `v0.11.0` | confirmed |
| DSN-02 | Surface overhauls | Landing, moderator console, participant surface, board, admin — one loop per surface, presentation only. | M11 | `v0.11.0` | confirmed |
| DSN-03 | Board appearance controls | High-contrast toggle, text/line sizing; responsive projector→TV→phone. | M11 | `v0.11.0` | confirmed |
| DSN-04 | Logo & color identity | Greenfield brainstorm at M11 kickoff. | M11 | `v0.11.0` | open (M11 kickoff) |
| DSN-05 | Board sound cue | Optional moderator-configured short sound on board devices when the line advances; board-side mute; MicLine-provided sound set; browser autoplay-unlock constraint noted. Decision to build finalizes at M11 kickoff. | M11 | `v0.11.0` | assumed |
| STD-10 | Event branding | Moderator logo + accent color on board/lobby/participant surface; the only image in the product (gated = accountable); specified and implemented with the M11 / `v0.11.0` design system. | M11 | `v0.11.0` | confirmed |

## M12 — Operational maturity and compliance

**Target release:** `v1.0.0`

Measured observability, cost modelling, budget alerts, load testing, operational audits, recovery runbooks, retention verification, and the compliance pack establish the first production-mature release.

| id | name | description | milestone | target version | status |
|---|---|---|---|---:|---|
| OPS-01 | Observability | Structured-log pipeline decision, usage dashboards, minimal alerting (keep-it-simple appetite). | M12 | `v1.0.0` | confirmed |
| OPS-02 | Cost model | Per-event/per-month formulas from measured usage; feeds admin € projection; ~€20/month busy-month target; alert-only budget threshold (notify, never auto-flip). | M12 | `v1.0.0` | confirmed |
| OPS-04 | Constitution audit & load test | Operator-burden audit → refactor backlog; scripted-client load test of the DO path (~500 participants); degraded modes designed only after this evidence. | M12 | `v1.0.0` | confirmed |
| OPS-05 | Rebuild-from-zero runbook | From nothing but the repo to a live PRD; re-opens the IaC question with data; documents Cloudflare-native emergency levers (WAF, Turnstile mode escalation) and the admin-wall reset procedure. | M12 | `v1.0.0` | confirmed |
| OPS-06 | Compliance pack | Lightweight ROPA, breach-response runbook (72 h duty, delete-all-data as last resort, Art.-34 notification procedure — moderators are the only stored data subjects — via admin export + external send), Cloudflare DPA check. | M12 | `v1.0.0` | confirmed |

## M13 — Monetization, if ever (PARKED)

**Target release:** *provisional v1.1.0*

The deliberately parked commercial layer: billing as another entitlement grantor, compensation policy, payment communications, and the German legal and tax package.

| id | name | description | milestone | target version | status |
|---|---|---|---|---:|---|
| BIL-01 | Billing as grantor | Provider (Stripe vs Merchant-of-Record) writes the same grant rows as invite codes; plans = named bundles kept active by subscription; no price/plan concepts anywhere else. Note: drastic admin controls (emergency stop, lockdown) need a compensation & communication policy toward paying customers. | M13 | *provisional `v1.1.0`* | confirmed (parked) |
| BIL-02 | German-operator legal pack | Impressum/DDG, USt/Kleinunternehmerregelung, AGB, Widerruf, invoicing — resolved with tax advisor/lawyer before charging anyone. Note: billing brings its own legally mandated transactional email set. | M13 | *provisional `v1.1.0`* | confirmed (parked) |

## Cross-milestone extensions

These features have one canonical introduction milestone and stable ID, while later milestones activate or add already-decided behavior.

| Feature | Introduced | Later extension |
|---|---:|---|
| FND-04 | M1 / `v0.1.0` | Admin UI wrapper in M6 / `v0.6.0`; invite-only control in M8 / `v0.8.0` |
| MOD-03 | M2 / `v0.2.0` | Purge-countdown states in M4 / `v0.4.0`; admin-removed state in M6 / `v0.6.0` |
| EVT-07 | M4 / `v0.4.0` | Emergency pause and resume grace in M7 / `v0.7.0` |
| AUTH-04 | M4 / `v0.4.0` | Non-empty-wallet carve-out activates in M8 / `v0.8.0` |
| PRT-08 | M4 / `v0.4.0` | Intake-closed state in M5 / `v0.5.0`; admin-ended state in M6 / `v0.6.0`; emergency state in M7 / `v0.7.0` |
| ADM-07 | M6 / `v0.6.0` | Invite-only-mode control activates in M8 / `v0.8.0` |
| ADM-11 | M6 / `v0.6.0` | Matrix extends to emergency controls in M7 / `v0.7.0` and entitlement controls in M8 / `v0.8.0` |
| OPS-03 | M4 / `v0.4.0` | Emergency scheduling behavior in M7 / `v0.7.0`; verification and monitoring audit in M12 / `v1.0.0` |

## Parked ideas (no milestone or target version; each with an unparking condition)

| id | name | description | milestone | target version | status |
|---|---|---|---|---|---|
| PRK-01 | Topics/tags | Structural steering ("questions for presenter B"); revisit at scale. | — | — | parked |
| PRK-02 | Live reactions | Board emoji bursts; possible per-event opt-in someday. | — | — | parked |
| PRK-03 | Participant-facing recap page | Killed for retention minimalism; export (STD-05) covers the need. | — | — | parked |
| PRK-04 | Waitlist for full events | Queue machinery we don't need twice. | — | — | parked |
| PRK-05 | In-product abuse report button | Email reports suffice at current scale. | — | — | parked |
| PRK-06 | Per-account retention dial | Fixed purge + delete-now covers the control need. | — | — | parked |
| PRK-07 | Vanity/NFC/other join methods | Beyond QR/URL/code/slug. | — | — | parked |
| PRK-08 | Admin UI v2 wishlist | Additional admin features beyond the M6 / `v0.6.0` operator-control-plane scope. | — | — | parked |
| PRK-09 | Clickable links in moderator content | Unparking requires a dedicated abuse/security review (QR-phishing chain). Until then all URLs render inert. | — | — | parked |
| PRK-10 | Images in moderator content | Unparking requires storage/purge, EXIF-privacy, and content-moderation-liability review. Branding logo (STD-10) stays the only image. | — | — | parked |
| PRK-11 | Request to speak (text-free line item) | Rejected for the original M1 scope (now M3 / `v0.3.0`); parked with trigger: real-event demand. Until then every line item requires written text. | — | — | parked |
| PRK-12 | Private moderator notes | Follow-up flag + decline reasons cover known jobs; trigger: concrete moderator demand after M9 / `v0.9.0`. | — | — | parked |
| PRK-13 | Browser push notifications | Strict unparking conditions: evidence participants background the tab + realistic iOS delivery path. Zero permission prompts until then. | — | — | parked |
| PRK-14 | Email-change flow | Delete + re-register for now; known pain: non-empty wallet. Revisit on real demand. | — | — | parked |

## Rejected in P2 (kept for context; do not resurrect without an explicit reopen)

| id | name | reason | target version | status |
|---|---|---|---|---|
| REJ-01 | Device fingerprinting / IP-identity binding | Venue-NAT blindness, collision false-positives on innocent participants, TDDDG/consent exposure, contradicts pillars 2+3 and the abuse doctrine. | — | rejected |
| REJ-02 | Board password | Reaffirmed rejected after explicit P2 challenge: can't protect content (phones stay zero-gate), punishes the moderator at the venue, opens the access-control door. Replaced by capability URL + regenerate link (MOD-07). | — | rejected |
| REJ-03 | Separate in-event information panel | Second content container; replaced by re-openable primer (STD-08). | — | rejected |
| REJ-04 | Waiting-time estimates | Variance-dominated; broken promises destroy trust. Positions, not prophecies. | — | rejected |
| REJ-05 | Participant-visible repeat badges & public timestamps | Public shaming chills participation; timestamp arithmetic weaponizes curated mode. Moderator-console-only instead. | — | rejected |
| REJ-06 | Reorder history & fairness warnings | Live visible motion is the fairness mechanism; ledgers have no consumer; nags moralize. | — | rejected |
| REJ-07 | Intake timers / auto-close | Surprise closure violates no-surprise moderation; intake close is manual only. | — | rejected |
| REJ-08 | Purge-warning / reminder / lifecycle emails | Anti-spam stance; in-product countdowns and transparency instead. | — | rejected |
| REJ-09 | Durable trails of moderator in-event actions | Audit log stays reserved for privileged admin/entitlement actions; hide/skip/announce leave no trail beyond purge-bound event state. | — | rejected |
| REJ-10 | Reschedule count cap | The ±24 h anchor to original values already bounds drift; a count cap punishes venue chaos. Rate limiting covers pathological churn. | — | rejected |
