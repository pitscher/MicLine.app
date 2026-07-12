# MicLine.app — Product Brief

> Provenance: P1 product-definition interview, 2026-07-06 · **P2 final product interview, 2026-07-12** (operator + Claude, per docs/build-guide/04-product-definition.md). Status: **v2 — final product foundation for the tech interview**. Companion documents: [feature-inventory.md](feature-inventory.md), [decision-log.md](decision-log.md). Technical decisions live in docs/inputs/tech-decisions.md until the tech interview supersedes it with docs/engineering/tech-context.md. Document language rule: English throughout; German terms appear only in the term sheet.

## One-liner

**MicLine is the live question line for every event:** hosts collect, order, and moderate audience questions in real time, so everyone in the room knows who gets the mic next.

## Positioning

MicLine's identity is the **live mic line**, not another Q&A board. The queue orders **people who will speak**, not just questions to be read aloud: the person marked "You're up" walks to the microphone, gets the mic passed to them, or is unmuted — and the people right behind them see their position and get mentally ready. (A moderator can always simply read a question aloud instead; that usage falls out for free.)

**Room-agnostic focus (P2):** in-room events remain the *design center* (dark hall, projector as join funnel), but **online and hybrid meetings are a fully supported usage context** — the zero-gate URL join works identically inside a video call. MicLine stays specific in its focus with deliberate versatility; no broadcast or livestream tooling follows from this.

### Pillars, in order

1. **The line** — a fair, transparent "everyone sees who's next". This is a category claim no incumbent (Slido, Mentimeter, Pigeonhole, Vevox) makes. First come, first served, positions visible to all.
2. **Zero friction** — QR scan, no account, no app install, no cookie banner. Participants are never asked for personal data.
3. **Privacy** — GDPR-clean by construction, EU-based operator, no trackers on any surface, radical data minimalism with short, honest retention.

**Simplicity** is a design principle rather than a marketing pillar: MicLine does one job — no polls, quizzes, word clouds, or slide decks, ever.

Open source is **not** a marketing pillar: the running SaaS must not link or refer to the public repository until the operator decides otherwise.

## What MicLine is not

- Not a livestream tool: no OBS overlays, no YouTube/Twitch chat ingestion (out of design scope for M1/M2).
- Not a classroom suite: classrooms work but are not marketed to.
- Not an engagement platform: no polls/quizzes/reactions/decks.
- Not a white-label product, not team/seat-based B2B software.
- Not a surveillance product: no participant fingerprinting, no tracking, no durable trails of moderator in-event actions.

## Audience & market

- **Primary:** in-room professional and community events, meetup-to-conference scale — meetups, panels, conference sessions, town halls, all-hands. Roughly 30–500 people in a room.
- **Supported context (P2):** online and hybrid meetings via URL join — same product, no integrations.
- **Tolerated, not targeted:** classrooms, workshops.
- **Out of scope (M1/M2):** livestreams/webinars at broadcast scale.
- **Buyer (eventually, M3-if-ever):** the individual moderator (prosumer, self-serve). B2B is "a bigger invite code sold manually over email" — never teams, seats, or org accounts in the data model.
- A conference with parallel tracks = separate events, one per room/session. No umbrella/series concept.

## Personas

| Persona | Who | What they need |
|---|---|---|
| **Moderator** | Creates and runs the event; the only authenticated human. Prosumer: meetup organizer, conference session host, works-council/town-hall facilitator. | Create an event in under a minute, share it, keep ground control of the line, never be surprised by the tool. |
| **Participant** | Audience member on their own device (phone-first, not phone-only). No account, no PII, possibly never asks anything. | See the board instantly after scanning, ask/withdraw a question, know exactly when they're up. |
| **Operator/Admin** | The owner (solo, Germany-based) wearing the admin hat. Day job exists; no pager. | Administer everything from the admin UI with zero code/dashboard/DB access (deliberate exception: admin identity itself is managed Cloudflare-side). Keep costs and abuse boringly under control. |

## The core model

### Event lifecycle

- A moderator creates an event **up to 48 h ahead** (configurable platform value). Until its start time it is **Scheduled**: visible only to the moderator, freely configurable.
- **Start and end date+time are required.** At start time the event opens automatically: participants can join, the line is live.
- Start and end are each adjustable within **±24 h of their originally configured values** — **any number of times within that window** (no count cap; the anchor to original values prevents drift). Rescheduling is a rate-limited action. Bigger moves require delete + recreate (anti-abuse guardrail; note: recreation mints a new join code).
- The moderator ends the event manually; a forgotten event **auto-ends at end_time + 24 h**. A live event is never killed by the platform for policy/config reasons — *committed events are sacred* (sole exceptions: the moderator's own end action, an admin force-end, the admin emergency stop; self-service account deletion requires ending a running event first).
- Settings freeze once the event is live (scheduling fields keep their ±24 h window; live controls such as announcements, intake close, and hide board are not settings).
- After the end: participants see a polite end screen ("join another event" path); the moderator sees an on-screen recap with an explicit content-deletion countdown; **event content is deleted 3 days after the end** (see Data lifecycle).

### The line — M1: Open line

- Mode **Open line** ("first come, first served"): a submitted question is appended directly to the public FIFO line. No pool, no reordering, no upvotes in M1.
- **Position stability is an open-line guarantee: your position only ever improves.** (Curated mode explicitly trades this guarantee for visible human ordering.)
- **One active waiting item per participant** — fixed platform rule, no per-event setting. Answered, skipped, or withdrawn frees the slot; then the participant may submit again. Enforcement is **session-scoped** (anonymous session cookie); the leave-and-rejoin residual is accepted as best-effort — device fingerprinting and IP-identity binding are **rejected** (venue-NAT blindness, fingerprint-collision false positives, consent/TDDDG exposure, pillar conflict). Marketing copy claims exactly "enforced per session", no more.
- **Every line item requires written text** (platform bounds 1–280 characters; per-event moderator-configurable min/max within those bounds). No text-free "request to speak" item type (parked).
- Full transparency: every participant sees the whole ordered line, "Now speaking" pinned on top.
- Moderator ground control: mark the top question **answered** (next is pulled automatically) or **skip** any item. **Skip removes the question text from all public surfaces immediately**; a minimal stub (attribution + "skipped") remains in the participant line; the projector board never shows skipped items; the owner keeps an honest "skipped" status chip. Both actions have a ~10 s single-level undo.
- Participants can **withdraw** their own not-yet-called question (slot frees instantly) and can **leave** the event (session reset; already-asked questions keep their original attribution).
- Position cues on the participant surface: sticky **"You're #N"** banner → **"You're up soon"** (approaching the top) → full-screen **"You're up!"**.
- No intake-closed state in M1: the event lifecycle is scheduled → live → ended (event-full and unknown-code are join screens, not lifecycle states). M1 workaround: announcement + manual end. Intake close ships in M2A.

### The line — M2B: Curated line

- Pool → moderator selects, orders, drag-reorders into the line; return-to-pool exists; skip works in both zones with the same stub semantics.
- **Pool visibility is a per-event creation setting:** a hidden pool auto-disables upvotes; a visible pool lets the moderator enable/disable upvotes. Both freeze at start. The owner **always** sees their own "in pool" chip regardless of visibility.
- Participant-side pool ordering: most-voted first when upvotes are on, newest first when off. The moderator's console sorts the pool freely.
- **Upvotes advise the human, never reorder the line.** Pool items only; nameless in both name modes; deduped per session (best-effort). Line items carry no vote UI and display no counts on participant surfaces or the board (the console may show counts to the moderator). No votes anywhere in open mode.
- **Review mode** (available in either line mode): submissions are held privately until approved. Open+review: approval **appends to the line at approval time** (approval order = line order; never retro-insertion above committed items). Curated+review: approval releases into the pool. Declined submissions: owner sees an honest "declined" chip with an **optional short moderator reason**; nobody else ever sees the item existed; declines are reversible until event end; retained until purge. With a hidden pool, review's participant-facing value is the explicit decline flow.
- **Duplicate merge** (pool-only): the moderator selects several pool items and picks a **primary**; the primary's text represents the group with a small "×N" badge (the badge travels with the primary into the line); merged items fold away publicly, their owners' chips show "merged" and inherit the primary's outcome; votes combine with one-voice-per-participant dedupe (best-effort); unmerge is possible until the group is consumed. When a merged question is called, the **primary's submitter** gets the mic. No merge machinery in open mode (the review tray declines duplicates there).
- **Per-event "max open submissions per participant"** in curated mode (counts active pool+line items; answered/skipped/withdrawn/declined/merged all free the slot). Default and range at M2B kickoff.
- **Co-moderators** (gateable): event-scoped helpers for review/curation. No hard dependency with review mode — solo review stays possible, with a consequence hint at creation ("on = you're the bottleneck").

### Announcements

- A live control, not a setting: **one current announcement**, replaces the previous, shown on the board's announcement area **and** as a banner on participant surfaces.
- **Bounded display lifetime** (platform config value; direction ~60 s): uniform auto-expiry on board *and* participant surfaces — no stale messages on unattended machines. The board shows a subtle self-dismissal cue (thin draining progress bar) so nobody thinks the board computer needs attention. The moderator can replace or **clear early**; re-sending restarts the lifetime. No history, no scheduling.
- **Division of labor:** the primer is the static reference (frozen at start); the announcement is the live channel.

### Names & anonymity

Per-event moderator choice at creation:
- **Nicknames (automatic)** — default. Every participant gets a stable, funny, localized pseudonym ("Curious Capybara"); the moderator can call it to the mic without anyone revealing identity.
- **Real names (required)** — participants are prompted for a name before their **first submission** (viewing stays zero-gate); the name is unverified, accepted as-is, and locked for the event. Typo escape hatch: leave + rejoin (with an explicit "you'll appear as ⟨Name⟩ for the whole event" confirm step).

Upvotes (M2B) are nameless in both modes.

### Joining

- Methods (all simultaneously valid, same 6-char code): **QR code** (share panel + projector board), **short URL** `micline.app/e/⟨CODE⟩`, **code box on the landing page**.
- **Zero-gate:** scanning lands directly on the live board. No account, no name prompt, no cookie banner, no CAPTCHA on entry (invisible Turnstile runs on first submission only). Anonymous session cookie, no PII.
- Pre-start: minimal lobby ("starts at ⟨time⟩"). Full event: clean "this event is full" screen (no waitlist). Ended/unknown code: polite notice with re-entry path.
- Optional moderator-configured **pre-join primer** (M1: plain text; M2B: markdown-lite). Per-event acknowledgment choice: **skippable** vs **required explicit confirmation** — and in the required mode the moderator can **customize the confirmation statement** (plain text, short platform-capped; localized default "I understand."). Purpose: a poke to actually read, explicitly not bulletproof. **The acknowledgment is a gate, not a record:** MicLine never stores who confirmed what, and it is not legal proof of consent — if recorded consent is needed, MicLine is the wrong tool.
- In M2B the primer becomes **re-openable in-event** (ⓘ sheet on the participant surface) — same content, no separate information panel. The primer never renders on the projector board.
- **All moderator-authored text renders inert** (primer, announcements, decline reasons, hide-board message, confirmation statements): URL-shaped text is never clickable or auto-linked; no images in moderator content. The M2B branding logo is the only image in the product (narrow, entitlement-gated, accountable).

### Accounts (moderators only)

- **Magic-link auth**: email is an identifier, never a credential. One email, one click, ~30-day session; unlimited events within it. No passwords, no profiles.
- Stored: email + optional display name + country (from Cloudflare request geolocation, for anonymous aggregate stats only). Nothing else, ever.
- **Self-service account deletion from M1.** If a live event is running, the flow requires ending it first (one tap away); scheduled events are listed in a quantified consequence preview and die with the account. The deletion UI is self-sufficiently final: loading state → explicit "all your data has been purged" confirmation. **No confirmation email follows** — the UI is the confirmation.
- Auto-purge after **1 month of inactivity** (no warning email — anti-spam stance) unless the account has a future event scheduled or a non-empty invite-code wallet. The auth screen carries a hint for returning-but-purged moderators.
- **No email-change flow** in current scope: delete + re-register (parked; the non-empty-wallet case is the known pain point).

### Entitlements & invite codes (M2A)

- **One entitlement seam:** anything gated is checked through a single capability interface. Grants come from the baseline, invite codes (vouchers), admin action — and possibly billing later, which would be *just another grantor* writing identical grant rows.
- **Free baseline (lockdown off):** every shipped feature, free. Fair-use limits: **100 participants/event**, **1 active + 1 planned event** per moderator. Invite codes raise limits (platform ceiling 1,000 participants/event) and unlock gateable capabilities.
- **Never gated:** the core Q&A mechanics — open line, curated line, review mode, timer, recap/export — **plus all P2 moderator tools:** intake close, hide board, regenerate board link, pool-visibility settings and limits, decline reasons, follow-up flag, console filters/search/bulk verbs. **Gateable:** co-moderators, event branding, vanity slugs, limit raises, registration permission.
- **Invite codes are bundles** `{may_register? + capability grants + limit raises + validity}` redeemed into an account wallet; codes can be multi-use (`max_redemptions`); two independent clocks (redeem-by; grant validity). Revocation and lockdown never touch a running or already-created event — they gate the *next* act (registration, event creation; duplicating an event is creation and therefore gated; **rescheduling a committed event is never gated**). A one-click **god voucher** preset grants everything, bounded only by redeem-by date and optional event meter.
- **Invite-only mode ("lockdown")**: flipped by the admin; new registrations *and* new event creation then require an invite code — existing events still run and start. Doubles as the cost brake.

## Surfaces

| Surface | Essence | Milestone |
|---|---|---|
| **Landing page** | Speaks to moderators (hero: the line pillar; 3-step how-it-works; create CTA), yet prominently funnels lost participants to join by code. Honest privacy block. Clean; depth lives in sub-pages. No pricing page, no repo link, no SLA disclaimers on the hero. | M1 (F007), beauty pass M4 |
| **Join flow** | QR/URL/code → board, zero-gate. Lobby / full / ended / unknown-code states. Optional primer with acknowledgment modes. | M1 (F005) |
| **Participant surface** | Device-agnostic (phone-first, dark-hall-first design center): Now speaking, full line with positions, own-question status chips, You're-#N → up-soon → You're-up cues, announcement banner, ask/withdraw/leave. M2B adds the pool zone (collapsible per viewer), upvotes, ⓘ primer sheet. | M1 (F005), M2B additions |
| **Projector board** | Read-only big-screen route behind its **own unguessable capability URL** (never exposes event ID or join code; moderator can **regenerate the link**). Join QR + code always visible, Now/Up-next huge, top of line, announcement area, fullscreen toggle. Never shows skipped items, timestamps, or vote counts. Can be **hidden** by the moderator (M2A): neutral customizable message, no line content, no QR while hidden. Per-viewer language and pool-collapse settings persist per device until event end. | M1 (F005), M2A hide, M4 overhaul |
| **Moderator console** | One screen, phone-capable: header (title, share panel, live participant count, end button), the line with answered/skip + undo, announcement composer, primer preview. M2A adds live controls (intake close, hide board, regenerate board link) and the follow-up flag; M2B adds pool/review tray, filters, search, bulk verbs (merge, bulk decline), submission timestamps and per-submitter counts. | M1 (F004), M2A/M2B additions |
| **Admin UI** | `/admin` behind a **dedicated mandatory auth wall** (one dedicated layer; mechanism decided in the tech interview; emergency reset always possible via the Cloudflare account). Oversight (search, god view incl. live states like board-hidden/intake-closed, force-end, soft-delete, ban, GDPR delete), metrics wall, global stats, platform controls, invite-code management, feedback-email setting. **Every admin control shows a quantified, timestamped consequence preview and defines its reverse action — nothing ever requires a redeploy to undo.** | M2A |
| **Transactional email** | See email inventory below. | M1/M2A |

## Transactional email — the complete inventory

| # | Trigger | Recipient | Milestone |
|---|---|---|---|
| E-1 | Magic-link login request | Moderator | M1 |
| E-2 | Feedback-box submission (delivery) | Operator (FEEDBACK_EMAIL) | M1 |
| E-3 | Admin force-ends an event (+ optional admin message) | Affected moderator | M2A |
| E-4 | Emergency stop activated (reason, implications, next steps; announces E-4b) | Moderators with running / scheduled / recently-ended events | M2A |
| E-4b | Emergency export delivery (all formats, ~24 h link, not-the-normal-process note, responsibility line) | Same cohort with exportable content | M2A |
| E-5 | Emergency stop reversed (resume; states purge-grace deadlines and late-open outcomes) | Same cohort | M2A |

**Principles:** transactional only, moderator's locale, short, zero marketing. Any new email type requires a product decision. **Deliberately no email for:** inactivity purge, 3-day content purge, account-deletion confirmation (the UI is the confirmation), event creation/reminders, auto-end, login alerts, grant expiry/code redemption (in-product transparency), service notices, bans (banned addresses silently stop receiving magic links — anti-enumeration-consistent), soft-deletes (in-product "removed by administration" state instead), export readiness, breach notification (M5 runbook procedure: admin export + external send). Co-moderator invitations are link-based, not email. M3 would bring its own legally mandated billing email set.

## Product principles

1. **The line is sacred:** fairness and transparency of who's next is the product. Nothing (votes, admin, entitlements) silently reorders or kills a committed event.
2. **Committed events are sacred:** lockdown, revocation, purges, and platform caution never break a running or scheduled event; fail closed for creation, fail open for running events. Sole termination paths: moderator end, admin force-end, emergency stop.
3. **Zero-gate participation:** watching costs nothing — no account, no PII, no banner, no browser-permission prompts. Friction may only ever guard *actions* (submit), never *presence*.
4. **Radical data minimalism, stated honestly:** collect the minimum, keep it briefly, delete on schedule, say so publicly. Marketing never claims more privacy than the code delivers. **Anonymous aggregates survive deletion; everything identifying dies.** No fingerprinting, no durable trails of moderator in-event actions.
5. **One entitlement seam** (future constitution article): gated code never knows where a grant came from.
6. **Solo-operator economics:** boring first-party primitives, no component that needs babysitting, best effort/no SLA/no pager; lockdown is the emergency brake for both abuse and cost.
7. **Invisible host:** on participant surfaces the event dominates; MicLine is a small wordmark. The landing page is where MicLine speaks.
8. **Not "yet another SaaS thing":** the landing page must earn a try or a bookmark; no buzzwords, no overload, sub-pages carry depth.
9. **Motion explains state** — position changes, the You're-up cue, and self-dismissal cues may move; nothing else does.
10. **Multilingual from the first commit** (en + de): every user-facing string flows through the i18n layer; adding a locale is a translation task, never a refactor.
11. **Phone-first, dark-hall-first, WCAG AA**, graceful on flaky venue Wi-Fi — and device-agnostic: "participant surface", not "phone".
12. **Races resolve into states, never errors:** when an interaction collides with a state change (ended, force-ended, emergency, intake closed, full), the surface transitions to the new canonical truth with calm feedback — no raw errors on participant surfaces.
13. **Spec-driven change** — no feature code outside the spec→plan→tasks loop.

## Limits & platform values (initial)

| Value | Baseline | Raisable to | Note |
|---|---|---|---|
| Participants per event | 100 | 1,000 (platform ceiling) | design/load-test envelope ~500; revisit after tech phase |
| Events per moderator | 1 active + 1 planned | via invite code | ended events unlimited (history purges anyway) |
| Create-ahead window | 48 h | — | configurable platform value |
| Reschedule window | ±24 h around original start/end | — | unlimited count within window; rate-limited action; beyond: delete + recreate |
| Question length (platform) | 1–280 chars | — | trim, collapse whitespace, strip control chars |
| Question length (per event) | within 1–280 | — | moderator-configurable min/max at creation |
| Active items per participant (open line) | 1 | — | fixed platform rule, session-scoped |
| Open submissions per participant (curated) | default at M2B kickoff | per-event setting | counts active pool+line items |
| Announcement display lifetime | ~60 s (direction) | — | uniform auto-expiry, all surfaces |
| Confirmation statement / hide-board message / decline reason | short caps | — | exact values at spec time |
| Emergency export link validity | ~24 h | — | platform config value |
| Post-resume purge grace | 24 h | — | overdue purges after emergency-stop reversal |
| Monthly cost target | ~€20 busy month | — | lockdown = cost brake; M5 alert-only budget threshold |

All such values live in a central, code-change-free platform configuration (mechanism decided in the tech interview), alongside a **global log-level control** (debug ↔ critical), freely adjustable by the operator.

## Data lifecycle (feeds privacy page & constitution Art. V)

| Datum | Lifetime |
|---|---|
| Magic-link token (hashed) | 15 min, single use |
| Moderator session | ~30 days |
| Event content (questions, names/pseudonyms, decline reasons, flags) | until **3 days after event end**, then purged (in-product countdown; no reminder email; purge jobs pause during an emergency stop, overdue purges run 24 h after resume) |
| Event row + stats | with the account |
| Moderator account (email, display name, country) | until self-deletion or **1 month inactivity** (carve-outs: future event scheduled; non-empty wallet) |
| Anonymous aggregates (counts, durations, country) | indefinitely — unlinkable by construction |
| Audit log (privileged admin/entitlement actions only — never moderator in-event actions) | 12 months |
| Participant data | none beyond the anonymous session cookie (which also carries participant UI language and per-device display preferences); nothing to delete |
| Exported files (CSV/JSON/HTML snapshot) | **outside MicLine's deletion domain** — the moderator's responsibility, stated honestly at export time and on the privacy page |

GDPR posture: Impressum + honest privacy page from day one; no third-party analytics/trackers; self-service delete (M1); M5 adds the operator compliance pack (lightweight ROPA, breach-response runbook incl. the delete-all-data control and the Art.-34 notification procedure, Cloudflare DPA check).

## Design direction (M4 seed — captured, not designed)

- **Adjectives:** calm · clear · even-handed · trustworthy · quietly confident. Playfulness only in participant micro-moments (pseudonyms, "You're up!"); console and admin stay all-business.
- **North-star reference: Luma** — uncluttered, instantly comprehensible, invites exploration, colors well used on a dark background. Anti-patterns: overloaded pages, too much text, register-modal ambushes. Anti-reference: Slido (and generic purple-gradient SaaS).
- **Dark mode default** (participant + board designed dark-first; board high-contrast, readable from the back row); light mode supported. Logo/colors: greenfield, M4 brainstorm.
- Board gets user-adjustable appearance (contrast, text/line sizing) in M4; responsive from projector to TV to phone. M4 also decides the **board sound cue** (optional moderator-configured sound on board devices when the line advances; board-side mute; MicLine-provided sound set; browser autoplay-unlock constraint noted).

## Language & tone

- Locales at launch: **en + de**. German uses **Du** by default; a **`de (formal)`** catalog (Sie) becomes selectable as *event language* in M2B, affecting participant surfaces + board only — the moderator's own UI stays Du.
- **Locale derivation (P2, supersedes the P1 model):** all of this governs UI chrome only — user-generated content is never translated.

| Surface | Default derived from | Adjustable | Persisted |
|---|---|---|---|
| Moderator console | Moderator device language | Anytime, by moderator | On the account, while it exists |
| Event language | Moderator device language at creation | Before event start only | On the event |
| Participant surface | **Participant device language** | Anytime, by participant | Session cookie / device (participants have no accounts) |
| Board | Configured event language | Anytime, per viewer (multiple viewers, independent choices) | Per device, until event end |

- Emails: moderator's persisted console language. Spec-time edges (named, deliberately not decided): device base-language vs event register variant (direction: the event's variant wins when bases match, e.g. device `de` + event `de (formal)` → Sie); one person holding participant surface + board (independent per-surface persistence).
- Register per surface: landing confident & curious, zero buzzwords · participant minimal words, warm micro-moments · console terse & operational · admin plain technical · email short, no marketing.
- **Failure-state tone rule:** calm, honest, no error codes or technical vocabulary on participant surfaces; failure copy always frames recovery, never termination.

### Term sheet (canonical vocabulary)

| Concept | EN | DE (Du default) |
|---|---|---|
| Container a moderator creates | event | Event |
| The queue | the line | die Reihe |
| M1 mode | Open line (first come, first served) | Offene Reihe |
| M2B mode | Curated line | Kuratierte Reihe |
| "You're up!" | You're up! | Du bist an der Reihe! |
| Removed by moderator | skipped | übersprungen |
| Name modes | Nicknames (automatic) / Real names (required) | Spitznamen (automatisch) / Klarnamen (erforderlich) |
| Voucher (user-facing) | invite code | Einladungscode |
| Lockdown (admin) | Invite-only mode | Einladung erforderlich |
| Emergency controls (admin) | Emergency stop / Delete all data | Notabschaltung / Alle Daten löschen |
| Event announcement | announcement | Mitteilung |
| Platform banner | service notice | Hinweis |
| Projector surface | the board | das Board |
| Board safety control | Hide board | Board verbergen |
| Pre-start event state | Scheduled | Geplant |
| Static export format (naming WIP) | Line snapshot (HTML) | — (at translation time) |

Untranslated new terms (intake close participant copy, review approve/decline verbs) are fixed at spec/translation time. URL scheme: `/e/⟨CODE⟩` (e = event) — confirmed. English documents use English terms; German appears only in this table.

## Operations posture

- **Best effort, no SLA, no pager** — placement rules: stated in terms + FAQ as what MicLine *does* ("built to reconnect gracefully; run with care by a small operator; no guaranteed availability"); never on the landing hero; never in-product during a live event. The FAQ carries the honest outage entry and points at the emergency machinery as the answer.
- **Submission & reconnect acceptance criteria (binding on the realtime spec):** ① no silent loss — a submission is durably saved and visible, or explicitly reported failed; ② retries never create duplicates; ③ explicit save states (sending / saved as #N / failed, tap to retry); ④ automatic recovery — identity, own items, chips, and position restore after reload/reconnect with no user action; "please refresh" is never the designed path.
- **Canonical failure states (closed list):** delayed save · failed save with retry · participant "reconnecting — the line continues" banner · stale-board chip (the unattended board self-describes lag) · event full · unknown/expired code — plus the state-takeover rule (principle 12) and the moderator-console set (reconnecting, action-failed-retry, admin-ended notice, emergency notice). Exact wording at spec time.
- **Degraded modes are M5-only**, designed after load-test evidence. Metrics visibility ladder: participants none, ever · moderators in-event product stats only (live participant count) · admin metrics wall · raw infrastructure operator-only.
- **Emergency stop model (M2A):** total stop — the platform freezes (events, participant surfaces, boards, console, moderator auth, creation, registration, all scheduled jobs including purges) with exactly three exceptions: static notice pages, the admin UI (reversal must never require a redeploy), and the expiring export-download endpoint. Sequence: freeze instantly → generate snapshot exports for affected events (running + ended-within-purge-window) → E-4 then E-4b emails. The trigger modal contains the export choice (**default ON**, "skip if you suspect data compromise"), a quantified impact preview ("X events: X running, X scheduled, X in export window"), and a computation timestamp. Scheduled events never open during a stop; on reversal they open late if their window is still current, otherwise they auto-end. Overdue purges run 24 h after resume; E-5 states the deadlines. Participants and boards show a calm emergency-ended screen — the sole participant-visible platform-initiated ending.
- **Emergency stop ≠ delete all data:** the stop is reversible quiescence; delete-all-data is the separate GDPR/breach nuke (step-up auth, typed confirmation — which the emergency stop now also requires — honest irreversibility labeling).

## Milestone frame (input to P4)

Execution order **M1 → M2A → M2B → M4 → M5 → M3-if-ever**.

- **M1 — Core product:** F000–F007, lean FIFO cut with the P2 semantic refinements (one-active-item, skip-stub, announcement lifetime, acknowledgment modes, min/max length, up-soon cue, locale model, failure states, deletion guard, log-level flag). No new surfaces added by P2.
- **M2A — Control plane & data:** entitlement engine → **recap & export** (CSV/JSON/HTML snapshot; the emergency machinery consumes it) → admin UI incl. the full emergency-stop model, force-end comms, emails E-3…E-5 → invite codes + lockdown; moderator live-safety controls (intake close, hide board, regenerate board link) and the follow-up flag slot in anywhere.
- **M2B — Stand-out modes (force-rank):** curated line (pool visibility, merge, limits, per-viewer pool toggles) → upvotes → review mode (decline reasons) → co-moderators — with console filters/search/bulk riding along — then speaker timer → duplicate/templates → rich announcements & primer (in-event ⓘ) → vanity slugs → branding (spec'd here, executes after M4). Parallel tracks: TMS, `de (formal)`.
- **M4 — UI overhaul:** design system first, then the four surfaces (presentation only); board appearance controls; board sound cue; logo & color identity.
- **M5 — Longevity:** observability, cost model (+ alert-only budget threshold), retention enforcement, constitution audit & load test, rebuild runbook (incl. Cloudflare-native emergency levers, admin-reset procedure, Art.-34 notification procedure), compliance pack.
- **M3-if-ever — Monetization (parked):** billing as just another grantor; German-operator legal pack; compensation & communication policy for drastic admin controls toward paying customers; legally mandated billing emails.

## Final product boundaries

- One event = one room/session; no presentation/session/umbrella/series/team/seat/org model.
- Participants never have accounts, never provide PII to watch, never see permission prompts; viewing is zero-gate — no board passwords, no company-domain gates, no participant invite codes.
- No per-question anonymity (event-level name modes only). No polls, quizzes, word clouds, reactions, slide decks, livestream ingestion, white-label, public directory, or waitlist.
- No fingerprinting or IP-identity binding, ever. No waiting-time estimates. No browser push notifications in current scope. No AI features in current scope (duplicate handling is human-controlled).
- Event content dies 3 days after event end; anonymous aggregates survive only if unlinkable; exports move responsibility to the moderator.
- The line is never silently reordered — by anyone or anything.
