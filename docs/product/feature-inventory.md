# MicLine.app — Feature Inventory

> Provenance: P1 interview 2026-07-06 · **P2 final product interview 2026-07-12**. Status values: **confirmed** (explicit interview decision) · **assumed** (Claude's call under interview rules, veto anytime) · **open** (deferred, owner noted) · **parked** (deliberately shelved) · **rejected** (decided against; kept for context). Milestones: **M1 → M2A → M2B → M4 → M5 → M3-if-ever**. IDs are stable from P1; new IDs exist only where P2 created genuinely new scope. M1 ids reference the F000–F007 feature set from docs/build-guide/06-m1-plan.md.

## M1 — Core product

| id | name | description | milestone | status |
|---|---|---|---|---|
| FND-01 | Monorepo & delivery pipeline | pnpm/turbo workspace, CI (lint/typecheck/test/build/secrets), DEV/PRD deploy via GitHub Actions only (F000). Tech interview must first outline the complete Cloudflare + GitHub setup; foundation is revisited once a bare-minimum deployable product exists. | M1 | confirmed |
| FND-02 | i18n plumbing | Lingui with `en` + `de`, every user-facing string extracted, unextracted strings fail CI (F000). Locale derivation model: see I18N-02. | M1 | confirmed |
| FND-03 | Security & hardening baseline | Security headers, WS origin allowlist, strict TS/zod boundaries, structured logs, audit-log table (privileged admin/entitlement actions only — never moderator in-event actions), soft-delete semantics, `users.role` column. | M1 | confirmed |
| FND-04 | Platform config & kill-switch flags | Central code-change-free config (retention windows, limits, P2 values: announcement lifetime, text caps, link validity, purge grace, curated participant-limit default) + KV flags: event-creation kill-switch, maintenance banner — wrangler CLI in M1, admin UI in M2A. | M1 | confirmed |
| FND-05 | Translation status tracking | Lunaria dashboard/PR-comments loop (EmDash pattern), non-gating tooling loop after F001. | M1 | confirmed |
| FND-06 | Global log-level control | Deployment-wide log level (debug ↔ critical), freely adjustable by the operator without code change; details at tech interview. | M1 | confirmed |
| EVT-01 | Event creation | Create up to 48 h ahead (configurable); required: title, start & end date+time; optional: private description, primer (+ acknowledgment mode + custom confirmation statement); settings: language, name mode, profanity filter, per-event min/max question length within platform 1–280 (F001). | M1 | confirmed |
| EVT-02 | Event scheduling lifecycle | Scheduled → auto-opens at start → ended manually or auto-end at end_time+24 h; start/end each adjustable ±24 h around original values, unlimited count within window (rate-limited action); bigger moves = delete + recreate (new join code). | M1 | confirmed |
| EVT-03 | Settings freeze | Event settings immutable once live (scheduling fields keep their ±24 h window; live controls — announcements, and from M2A intake close / hide board / hide-message — are not settings). | M1 | confirmed |
| EVT-04 | Join codes & share panel | 6-char Crockford code, `/e/⟨CODE⟩` URL, QR; share panel in console (F001). | M1 | confirmed |
| EVT-05 | End-of-event recap (on-screen) | Moderator sees stats recap after end with explicit content-deletion countdown; stats (survive) visually separated from content (dies); participants get polite end screen with "join another event" path. | M1 | confirmed |
| EVT-06 | Event stats aggregates | Anonymous aggregates (counts, duration, country) snapshot at end; survive all deletions unlinkably. | M1 | confirmed |
| EVT-07 | 3-day content retention | Event content purged 3 days after end; in-product countdown, no reminder email; purge jobs pause during an emergency stop, overdue purges run 24 h after resume (export ships M2A). | M1 | confirmed |
| AUTH-01 | Magic-link auth | Email-only moderator auth (E-1): hashed single-use token (15 min), ~30-day session cookie, anti-enumeration; banned addresses silently stop receiving links (F002). | M1 | confirmed |
| AUTH-02 | Durable-minimal accounts | Stored: email, optional display name, country (CF geolocation, aggregates only), persisted console language; nothing else. | M1 | confirmed |
| AUTH-03 | Self-service account deletion | "Delete my account and data" from day one (GDPR Art. 17). Running event must be ended first; scheduled events listed in quantified consequence preview; UI is self-sufficiently final ("all your data has been purged"), no confirmation email. | M1 | confirmed |
| AUTH-04 | Inactivity auto-purge | Account purged after 1 month inactivity, no warning email; carve-outs: future event scheduled or non-empty wallet; re-registration hint on auth screen. | M1 | confirmed |
| LINE-01 | Open line (FIFO) | Submitted question appended straight to the public line; order = arrival; only M1 mode. Position stability ("your position only ever improves") is an open-line guarantee. | M1 | confirmed |
| LINE-02 | Full-line transparency | All participants see the whole ordered line + Now speaking; positions visible to everyone. | M1 | confirmed |
| LINE-03 | Moderator ground control | Mark top question answered (next auto-pulled); skip any item — skip removes the text from all public surfaces, a stub (attribution + "skipped") remains in the participant line, the board never shows skipped items, the owner keeps a "skipped" chip. | M1 | confirmed |
| LINE-04 | Undo | ~10 s single-level undo after answered and skip. | M1 | confirmed |
| LINE-05 | Withdraw own question | Participant removes their own not-yet-called question; slot frees instantly. | M1 | confirmed |
| LINE-06 | Position banner & cues | Sticky "You're #N" → "You're up soon" (approaching the top) → full-screen "You're up!" on the participant surface. | M1 | confirmed |
| LINE-07 | Realtime engine | One Durable Object per event, WebSocket hibernation, snapshot+deltas, state-machine invariants, graceful reconnect (F003). Binding acceptance criteria: no silent loss · duplicate-free retries · explicit save states · automatic recovery (no manual refresh as designed path). | M1 | confirmed |
| LINE-08 | One active question per participant | Open line: exactly one active waiting item per participant (fixed platform rule); answered/skipped/withdrawn frees the slot. Enforcement session-scoped; leave-and-rejoin residual accepted as best-effort (fingerprinting/IP binding rejected); copy claims "enforced per session". | M1 | confirmed |
| PRT-01 | Zero-gate join | QR/URL/code → live board instantly; no account/name/cookie-banner/CAPTCHA/permission prompts on entry; anonymous session cookie. | M1 | confirmed |
| PRT-02 | Join methods | QR + short URL + landing-page code box, all simultaneously valid. | M1 | confirmed |
| PRT-03 | Name modes | Per-event: Nicknames (automatic, localized funny pseudonyms, default) or Real names (required before first submission, locked for the event, unverified). | M1 | confirmed |
| PRT-04 | Name confirm & leave/rejoin | Explicit "you'll appear as ⟨Name⟩" confirm; Leave-event resets session (questions keep old attribution; Turnstile re-runs). | M1 | confirmed |
| PRT-05 | Pre-join primer & acknowledgment | Optional plain-text briefing before joining; per-event choice: skippable vs required explicit confirmation with moderator-customizable statement (plain text, capped; localized default "I understand."). Acknowledgment is a gate, not a record — nothing stored per participant, not legal proof. | M1 | confirmed |
| PRT-06 | Join-state screens | Lobby ("starts at ⟨time⟩"), event-full (no waitlist), ended, unknown/expired code. | M1 | confirmed |
| PRT-07 | Own-question status | Chips: in line #N / answered / skipped (M2B adds: in pool / declined / merged); withdrawn disappears. | M1 | confirmed |
| PRT-08 | Failure states & state-takeover | Canonical closed list: delayed save, failed save + retry, "reconnecting — the line continues" banner, stale-board chip, event full, unknown code. Races resolve into the new canonical state with calm feedback, never raw errors. Console set: reconnecting, action-failed-retry, admin-ended notice, emergency notice. Tone: calm, honest, no error codes on participant surfaces. Device-agnostic ("participant surface", not phone). | M1 | confirmed |
| BRD-01 | Projector board (minimal) | Read-only big-screen route behind its own unguessable capability URL (never exposes event ID or join code): QR + code corner, Now/Up-next huge, top of line, announcement area, fullscreen toggle; one layout, no settings; never shows skipped items, timestamps, or vote counts. | M1 | confirmed |
| MOD-01 | Moderator console | One phone-capable screen: header (title, share panel, live participant count, end button), the line with actions, announcement composer (F004). | M1 | confirmed |
| MOD-02 | Announcements | Live control: one current announcement, replaces previous, shown on board + participant surfaces; bounded display lifetime (platform value, ~60 s direction) with uniform auto-expiry everywhere; subtle draining progress cue on the board (style assumed); clear-early action; re-send restarts; no history/scheduling. | M1 | confirmed |
| MOD-03 | My-events list | Dashboard: scheduled / active / past events (past until purge, with deletion countdown; admin-removed events show "removed by administration"), sorted by date. | M1 | confirmed |
| MOD-04 | Feedback funnel (minimal) | Optional post-event feedback box; delivered by email (E-2) to FEEDBACK_EMAIL (Wrangler secret in M1; admin-UI setting in M2A). | M1 | confirmed |
| SAFE-01 | Bot defense | Invisible Turnstile on: auth request, event creation, first submission per participant session. | M1 | confirmed |
| SAFE-02 | Rate limiting | Per-action limits keyed ip/uid/sid (Workers binding or KV fallback); rescheduling is a rate-limited action. | M1 | confirmed |
| SAFE-03 | Content rules | Trim, platform 1–280 chars (per-event min/max within bounds via EVT-01), collapse whitespace, strip control chars. All moderator-authored text renders inert (no clickable/auto-linked URLs, no images). | M1 | confirmed |
| SAFE-04 | Profanity filter | Optional per-event wordlist filter, default off (F006). | M1 | confirmed |
| MKT-01 | Landing page | Moderator-pitch hero (line pillar), 3-step how-it-works, create CTA, prominent join-code box, honest privacy block, visual line mock; clean — depth in sub-pages; no pricing, no repo link, no SLA disclaimers on the hero. | M1 | confirmed |
| MKT-02 | Sub-pages | How it works, FAQ (incl. honest outage entry), privacy (truthful lifetimes/deletion/export responsibility), terms (best-effort/no-SLA phrased as what MicLine does), Impressum, contact (F007). | M1 | confirmed |
| I18N-02 | Surface locale derivation | Console: device default, adjustable anytime, persisted on account · Event language: device default at creation, adjustable before start only · Participant surface: participant device default, adjustable anytime, persisted in session/device · Board: event-language default, per-viewer adjustable, persisted per device until event end · Emails: moderator's console language. UI chrome only, never content. Spec edges: register-variant resolution, dual-surface viewer. | M1 | confirmed |

## M2A — Control plane & data (sequence: engine → export → admin incl. emergency → codes/lockdown; live controls slot in anywhere)

| id | name | description | milestone | status |
|---|---|---|---|---|
| ENT-01 | Entitlement engine | Single `can(actor, capability, ctx)` seam; grants table with source (baseline/invite-code/admin/billing-future), scope, expiry, revocation; audit-logged. | M2A | confirmed |
| ENT-02 | Free baseline | All features free (lockdown off); limits 100 participants/event, 1 active + 1 planned event. Never-gated: core Q&A mechanics + all P2 moderator tools (intake close, hide board, regenerate board link, pool settings/limits, decline reasons, follow-up flag, filters/search/bulk, recap/export). Gateable: co-mods, branding, vanity slugs, limit raises, registration. | M2A | confirmed |
| ENT-03 | Invite codes (vouchers) | Bundles {may_register? + grants + limit raises + validity} redeemed into account wallet; multi-use via max_redemptions; two clocks (redeem-by, grant validity); god-voucher preset; design details (format, QR, custom message, generation UX, clock defaults) at M2A kickoff. | M2A | confirmed |
| ENT-04 | Revocation semantics | Kill unredeemed code or revoke redeemed grant; effect at next gated act; never touches running events; rescheduling committed events is never a gated act. | M2A | confirmed |
| ENT-05 | Invite-only mode (lockdown) | Gates new registration and new event creation (incl. duplicate-event) behind invite codes; existing events run/start normally; doubles as cost brake. | M2A | confirmed |
| ENT-06 | Entitlement transparency | Account page shows active grants, meters remaining, validity. | M2A | confirmed |
| ADM-01 | Admin bootstrap | Admin role exclusively via ADMIN_EMAILS Wrangler secret (Cloudflare-side); no in-product admin management; recovery = update secret. | M2A | confirmed |
| ADM-02 | Admin auth wall | /admin behind one dedicated mandatory auth layer (not magic-link + stacked factor). Mechanism (Cloudflare Access vs TOTP vs passkey) decided at tech interview with pricing/limits/exit-cost criteria; hard requirement: emergency reset always possible via Cloudflare account access alone — no lockout dead-end. Step-up confirmation on destructive actions regardless. | M2A | confirmed (mechanism: open → tech interview) |
| ADM-03 | Oversight & abuse tools | Search events/moderators, detail views, read-only god view of live events incl. live states (board hidden, intake closed), force-end (calm outside: participants/board see normal ended state; truthful inside: moderator sees "ended by platform administration" + contact path + optional admin message + email E-3), soft-delete (owner sees "removed by administration"), ban moderator email (silent), GDPR full delete. | M2A | confirmed |
| ADM-04 | Abuse doctrine | Participants = moderator's problem (skip-stub, hide board, primer, filters, rate limits); moderators/events = admin's; reports via contact email, no in-product report button. | M2A | confirmed |
| ADM-05 | Metrics wall | Own-counter stats: events (created/active/week), participants, questions, emails sent, Turnstile failures, rate-limit hits, storage growth; € projection arrives with M5 cost model. | M2A | confirmed |
| ADM-06 | Global stats view | Totals + anonymous aggregates: events hosted/planned, participants approx., moderator countries, event durations. | M2A | confirmed |
| ADM-07 | Platform controls | Service notice banner (placement: always landing/auth/console; participant surfaces + board only when flagged as affecting live events), creation kill-switch (UI over M1 flags), invite-only mode toggle. | M2A | confirmed |
| ADM-08 | Emergency stop | Total stop with typed confirmation + step-up: ends running events, blocks creation and registration, freezes all surfaces, moderator auth, and scheduled jobs incl. purges. Exactly three exceptions: static notice pages, admin UI (reversal), expiring export-download endpoint. Sequence: freeze → snapshot exports for affected events → emails E-4/E-4b (trigger-modal export choice, default ON, "skip if you suspect data compromise"). Scheduled events never open during a stop; on reversal open late if window current, else auto-end; overdue purges 24 h after resume; E-5 states deadlines. Participants/board: calm emergency-ended screen. | M2A | confirmed |
| ADM-09 | Delete all data | Separate GDPR/breach nuke: step-up auth, typed confirmation, consequence preview, honest irreversibility labeling; D1 time-travel ≤30 days is an infra safety net, DO storage is not; operationally preceded by emergency stop (runbook). | M2A | confirmed |
| ADM-10 | Feedback email setting | Admin-configurable dedicated feedback address (takes over from M1 secret). | M2A | confirmed |
| ADM-11 | Consequence previews & reversal matrix | Every admin control shows a quantified consequence preview ("X events affected: X running, X scheduled, X in export window") with computation timestamp ("computed at ⟨time⟩ — may already be outdated") and defines its reverse action in the UI; nothing requires a redeploy to undo; irreversible parts labeled honestly. | M2A | confirmed |
| STD-05 | Recap & export | Rank 1 (raised from 5; closes OQ5): post-end recap + one-click export in CSV, JSON, and Line snapshot (HTML — self-contained one-pager mirroring the queue incl. pool; naming WIP). Contents: everything (questions, attribution, times, final statuses incl. declined + reasons, merge structure, votes, follow-up flags, primer, stats). No export-time filters. Responsibility line at export + privacy page. Entitlement-independent, never gated. Format list re-validated at M2A kickoff. | M2A | confirmed |
| MOD-05 | Intake close | Moderator live control, manual toggle only (no timers/auto-close), freely reversible, works in both line modes (stops new submissions/pool intake); line stays visible and withdrawable; honest "the line is closed for new questions" copy; display details at spec. | M2A | confirmed |
| MOD-06 | Hide board | Moderator live control: projector board shows event title + neutral message (system default, moderator-customizable at hide-time, platform cap) + wordmark; no line content, no QR/code while hidden. Board-only — participant surfaces and intake unaffected. No audit-log entry (ops logs only). Distinct from admin emergency stop. | M2A | confirmed |
| MOD-07 | Regenerate board link | Board route uses its own unguessable capability token (never event ID/join code; no enumeration — tech requirement); moderator can invalidate and re-mint the board URL anytime (leak recovery). Adopted instead of the rejected board password. | M2A | confirmed |
| MOD-08 | Follow-up flag | Moderator-only boolean on any item (pool/line/answered/skipped), set during event or recap window; never visible to participants, never affects public state; export column; dies with purge. | M2A | confirmed |

## M2B — Stand-out modes (force-rank: curated → upvotes → review → co-mods → timer → templates → rich content → vanity → branding; parallel: TMS, de-formal)

| id | name | description | milestone | status |
|---|---|---|---|---|
| STD-01 | Curated line mode | Pool → moderator selects/orders/drag-reorders into the line; return-to-pool; skip-stub everywhere; duplicate merge (pool-only: moderator picks primary, ×N badge travels with it, merged items inherit primary's outcome, votes combine deduped, unmerge until consumed, primary's submitter gets the mic). Pool visibility per-event setting (hidden auto-disables upvotes); owner always sees own chip. Per-event max open submissions per participant (pool+line; terminal states free the slot). Defaults (pool visible, upvotes on) assumed — finalize at M2B kickoff. | M2B | confirmed |
| STD-02 | Upvotes + pool sort | Participants upvote pool questions (nameless, session-deduped best-effort); participant pool sort votes-desc (or newest when off); votes advise curation, never reorder the line; no vote UI/counts on line items on participant surfaces or board; requires visible pool. | M2B | confirmed |
| STD-03 | Review mode (pre-moderation) | Submissions need approval before becoming public, in either mode; open+review appends at approval time (approval order = line order); curated+review releases into pool; declined = owner-visible chip + optional short moderator reason (plain text, capped, inert), reversible until event end, never public, retained until purge; consequence hints at creation; no co-mod dependency. | M2B | confirmed |
| STD-04 | Co-moderators | Invite helpers to review/curate simultaneously (event-scoped role); gateable; invitations link-based (no platform email) — mechanism at M2B kickoff. | M2B | confirmed |
| STD-06 | Speaker timer | Elapsed time on Now card, optional soft countdown on the board. | M2B | confirmed |
| STD-07 | Duplicate event → templates | One-click re-create with same settings (creation act — gated under lockdown); saved presets later. | M2B | confirmed |
| STD-08 | Rich announcements & primer | Markdown-lite (structure only — subset at M2B kickoff; no links rendered clickable, no images) for primer + announcements; primer becomes re-openable in-event (ⓘ sheet on participant surface) — one container, no separate info panel. | M2B | confirmed |
| STD-09 | Vanity join slugs | Custom slug alongside the code; gateable perk. | M2B | confirmed |
| STD-10 | Event branding | Moderator logo + accent color on board/lobby/participant surface; the only image in the product (gated = accountable); spec'd in M2B, executes after M4 design system. | M2B | confirmed |
| STD-11 | Rented TMS pipeline | Parallel track: Weblate/Crowdin (free OSS tier) syncing catalogs via PRs; replaces-or-feeds Lunaria — decided at M2B kickoff. | M2B | confirmed |
| MOD-09 | Console filters, search & bulk verbs | Minimal set: pool sort (votes/newest/oldest), review-tray views (pending/approved/declined), flagged-only view, per-submitter counts + submission timestamps (console-only), text search over pool/line/declined, multi-select with exactly two bulk verbs (merge, bulk decline — no bulk-approve). Full cross-surface filter matrix (incl. filter-by-submitter, participant/board display filters) at M2B kickoff with transparency guardrail: full line is always the default view; the board keeps one canonical layout. | M2B | confirmed |
| PRT-09 | Per-viewer pool collapse | Participants and board viewers can collapse/expand the pool zone per device (persisted until event end); the line itself is never hideable; defaults (direction: pool collapsed on the board) at M2B kickoff. | M2B | confirmed |
| I18N-01 | de (formal) event language | Second German catalog (Sie) selectable as event language; participant surfaces + board only, moderator UI stays Du; register-variant edge (device `de` + event `de (formal)` → Sie direction) resolved at spec time. | M2B | assumed |

## M3 — Monetization (PARKED; runs last: M1→M2A→M2B→M4→M5→M3-if-ever)

| id | name | description | milestone | status |
|---|---|---|---|---|
| BIL-01 | Billing as grantor | Provider (Stripe vs Merchant-of-Record) writes the same grant rows as invite codes; plans = named bundles kept active by subscription; no price/plan concepts anywhere else. Note: drastic admin controls (emergency stop, lockdown) need a compensation & communication policy toward paying customers. | M3 | confirmed (parked) |
| BIL-02 | German-operator legal pack | Impressum/DDG, USt/Kleinunternehmerregelung, AGB, Widerruf, invoicing — resolved with tax advisor/lawyer before charging anyone. Note: billing brings its own legally mandated transactional email set. | M3 | confirmed (parked) |

## M4 — UI overhaul (design system first, then surfaces)

| id | name | description | milestone | status |
|---|---|---|---|---|
| DSN-01 | Design brief & system | Design interview building on captured direction (Luma north star, adjectives, dark-first, motion-explains-state, invisible host); tokens/components in packages/ui. | M4 | confirmed |
| DSN-02 | Surface overhauls | Landing, moderator console, participant surface, board, admin — one loop per surface, presentation only. | M4 | confirmed |
| DSN-03 | Board appearance controls | High-contrast toggle, text/line sizing; responsive projector→TV→phone. | M4 | confirmed |
| DSN-04 | Logo & color identity | Greenfield brainstorm at M4 kickoff. | M4 | open (M4 kickoff) |
| DSN-05 | Board sound cue | Optional moderator-configured short sound on board devices when the line advances; board-side mute; MicLine-provided sound set; browser autoplay-unlock constraint noted. Decision to build finalizes at M4 kickoff. | M4 | assumed |

## M5 — Longevity

| id | name | description | milestone | status |
|---|---|---|---|---|
| OPS-01 | Observability | Structured-log pipeline decision, usage dashboards, minimal alerting (keep-it-simple appetite). | M5 | confirmed |
| OPS-02 | Cost model | Per-event/per-month formulas from measured usage; feeds admin € projection; ~€20/month busy-month target; alert-only budget threshold (notify, never auto-flip). | M5 | confirmed |
| OPS-03 | Retention enforcement | Automated purge jobs for every lifetime in the data-lifecycle table (DO cleanup via alarms; emergency-stop pause + 24 h resume grace). | M5 | confirmed |
| OPS-04 | Constitution audit & load test | Operator-burden audit → refactor backlog; scripted-client load test of the DO path (~500 participants); degraded modes designed only after this evidence. | M5 | confirmed |
| OPS-05 | Rebuild-from-zero runbook | From nothing but the repo to a live PRD; re-opens the IaC question with data; documents Cloudflare-native emergency levers (WAF, Turnstile mode escalation) and the admin-wall reset procedure. | M5 | confirmed |
| OPS-06 | Compliance pack | Lightweight ROPA, breach-response runbook (72 h duty, delete-all-data as last resort, Art.-34 notification procedure — moderators are the only stored data subjects — via admin export + external send), Cloudflare DPA check. | M5 | confirmed |

## Parked ideas (no milestone; each with an unparking condition)

| id | name | description | milestone | status |
|---|---|---|---|---|
| PRK-01 | Topics/tags | Structural steering ("questions for presenter B"); revisit at scale. | — | parked |
| PRK-02 | Live reactions | Board emoji bursts; possible per-event opt-in someday. | — | parked |
| PRK-03 | Participant-facing recap page | Killed for retention minimalism; export (STD-05) covers the need. | — | parked |
| PRK-04 | Waitlist for full events | Queue machinery we don't need twice. | — | parked |
| PRK-05 | In-product abuse report button | Email reports suffice at current scale. | — | parked |
| PRK-06 | Per-account retention dial | Fixed purge + delete-now covers the control need. | — | parked |
| PRK-07 | Vanity/NFC/other join methods | Beyond QR/URL/code/slug. | — | parked |
| PRK-08 | Admin UI v2 wishlist | Additional admin features beyond M2A scope. | — | parked |
| PRK-09 | Clickable links in moderator content | Unparking requires a dedicated abuse/security review (QR-phishing chain). Until then all URLs render inert. | — | parked |
| PRK-10 | Images in moderator content | Unparking requires storage/purge, EXIF-privacy, and content-moderation-liability review. Branding logo (STD-10) stays the only image. | — | parked |
| PRK-11 | Request to speak (text-free line item) | Rejected for M1; parked with trigger: real-event demand. Until then every line item requires written text. | — | parked |
| PRK-12 | Private moderator notes | Follow-up flag + decline reasons cover known jobs; trigger: concrete moderator demand after M2. | — | parked |
| PRK-13 | Browser push notifications | Strict unparking conditions: evidence participants background the tab + realistic iOS delivery path. Zero permission prompts until then. | — | parked |
| PRK-14 | Email-change flow | Delete + re-register for now; known pain: non-empty wallet. Revisit on real demand. | — | parked |

## Rejected in P2 (kept for context; do not resurrect without an explicit reopen)

| id | name | reason | status |
|---|---|---|---|
| REJ-01 | Device fingerprinting / IP-identity binding | Venue-NAT blindness, collision false-positives on innocent participants, TDDDG/consent exposure, contradicts pillars 2+3 and the abuse doctrine. | rejected |
| REJ-02 | Board password | Reaffirmed rejected after explicit P2 challenge: can't protect content (phones stay zero-gate), punishes the moderator at the venue, opens the access-control door. Replaced by capability URL + regenerate link (MOD-07). | rejected |
| REJ-03 | Separate in-event information panel | Second content container; replaced by re-openable primer (STD-08). | rejected |
| REJ-04 | Waiting-time estimates | Variance-dominated; broken promises destroy trust. Positions, not prophecies. | rejected |
| REJ-05 | Participant-visible repeat badges & public timestamps | Public shaming chills participation; timestamp arithmetic weaponizes curated mode. Moderator-console-only instead. | rejected |
| REJ-06 | Reorder history & fairness warnings | Live visible motion is the fairness mechanism; ledgers have no consumer; nags moralize. | rejected |
| REJ-07 | Intake timers / auto-close | Surprise closure violates no-surprise moderation; intake close is manual only. | rejected |
| REJ-08 | Purge-warning / reminder / lifecycle emails | Anti-spam stance; in-product countdowns and transparency instead. | rejected |
| REJ-09 | Durable trails of moderator in-event actions | Audit log stays reserved for privileged admin/entitlement actions; hide/skip/announce leave no trail beyond purge-bound event state. | rejected |
| REJ-10 | Reschedule count cap | The ±24 h anchor to original values already bounds drift; a count cap punishes venue chaos. Rate limiting covers pathological churn. | rejected |
