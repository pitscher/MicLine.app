# MicLine.app — Constitution input

> Provenance: P2 tech interview, 2026-07-14 (operator + Claude, per docs/build-guide/04-product-definition.md §4) · additional-features review, 2026-07-15 (decision-log §AF — admin-wall model, export inertness, default-deny indexing, bounded inputs). Purpose: the enduring **principles and constraints** feeding prompt P3 (`/speckit.constitution`). Deliberately contains **no technology choices, no schemas, no mechanisms** — those live in docs/engineering/tech-context.md and may change; everything below is meant to survive any implementation swap. Sources: product-brief.md "Product principles", decision-log (P2/P3/FR), and the tech-interview decisions where they hardened a principle into a constraint.

## I. Public repository, zero secrets

- No secret, credential, token, key, or personal data (including real email addresses) in any tracked file, commit message, log statement, or test fixture — ever. Secrets exist only in gitignored local files, the platform's secret store, and CI environment secrets scoped per deployment environment.
- The example secrets file documents keys with **no values**. Secret generation commands may be documented; values never.
- Automated secret scanning gates every commit and every CI run; a leak is treated as an incident (rotate, then scrub), never as a cleanup chore.

## II. Cloudflare-first, seams for everything external

- All compute, storage, delivery, email, and bot defense run on Cloudflare primitives. External third-party HTTP APIs are permitted **only** where no Cloudflare primitive exists, must sit behind a service-layer seam, and require a decision-log entry.
- Even first-party services with churn risk (email sending) are consumed through a single service seam so the provider is swappable without touching callers.
- The entire infrastructure must be recreatable from the repository alone plus a short, logged list of manual console steps. Every manual click-step is logged the day it happens.

## III. Built to run for years by one person

- Prefer boring, managed, first-party primitives; minimize dependencies; no component that needs babysitting, no standing servers, no pager.
- Every feature plan must answer: *what does this cost the operator per month in money and attention?*
- Best effort, no SLA — stated where the product speaks about itself (terms/FAQ), never on the landing hero, never in-product mid-event.
- Lockdown (invite-only mode) is the emergency brake for both abuse and cost; alert thresholds notify, they never auto-flip anything.

## IV. Realtime correctness — the line state machine (two-mode-shaped)

- One component is the **single authority** for live event state. All mutations serialize through it; clients render server state and never compute authoritative state. Role is re-checked at the authority on every mutating message, regardless of what the edge asserted.
- **Invariants, enforced at the authority and covered by tests:**
  - At most one "Now speaking" per event.
  - Open line: exactly one active waiting item per participant (session-scoped; freed by answered/skipped/withdrawn).
  - **The line is never silently reordered** — by votes, admin action, entitlements, or any hidden logic.
  - Open-line position stability: a participant's position only ever improves. (Curated mode — M9 — visibly trades this for human ordering; the trade is explicit, never silent.)
  - Skip-stub visibility: skipping removes question **text** from all public surfaces immediately; an attribution + "skipped" stub remains in the participant line; the projector board never shows skipped items; the owner keeps an honest status chip.
  - Terminal states are immutable (short single-level undo windows excepted, as designed).
  - Once M9 exists: votes advise the human and never reorder the line; vote dedupe is per session, best-effort; upvotes are nameless in both name modes.
- The domain model is **two-mode-shaped from the first schema**: pool / review / declined / merged states and curated ordering must slot in without a rewrite — but no curated machinery ships before its milestone.
- **Binding reconnect/submission acceptance criteria (T9-2, verbatim):** ① no silent loss — a submission is durably saved and visible, or explicitly reported failed; ② retries never create duplicates; ③ explicit save states (sending / saved as #N / failed, tap to retry); ④ automatic recovery — identity, own items, chips, and position restore after reload/reconnect with no user action; "please refresh" is never the designed path. If an implementation can't meet one, the implementation changes, not the promise.
- **Races resolve into states, never errors:** when an interaction collides with a state change (ended, force-ended, emergency, intake closed, full), surfaces transition to the new canonical truth with calm feedback — no raw errors on participant surfaces.

## V. Privacy by default (operator in Germany; GDPR applies)

- **Zero-gate participation:** watching costs nothing — no account, no PII, no cookie banner, no CAPTCHA, no browser-permission prompts on entry. Friction may only ever guard *actions* (submit), never *presence*.
- Participants have **no accounts and no global identifier**: participant identity is anonymous, event-scoped by construction, and cross-event correlation is structurally impossible — "participants: nothing to delete" must be true by architecture, not by policy.
- **No fingerprinting, no IP-identity binding, ever** (REJ-01). Enforcement of participant limits is session-scoped best-effort; the leave-and-rejoin residual is accepted. Copy claims exactly "enforced per session", no more.
- **The acknowledgment is a gate, not a record:** the product never stores who confirmed what; primer acknowledgment leaves zero server-side trace.
- Moderator data is the decided minimal set (email, optional display name, country for aggregates, console language, plus unavoidable operational state); every stored datum has a defined lifetime and exactly one deletion path — the product brief's data-lifecycle table is **binding**.
- **Anonymous aggregates survive deletion only if unlinkable by construction** (no keys back to any person or event); everything identifying dies on schedule.
- **Deletion honesty:** privacy claims must reflect storage reality, including infrastructure-level point-in-time-recovery windows on the global database. Marketing and legal pages never claim more privacy or more deletion than the platform delivers. Exports leave the deletion domain — stated honestly at export time and on the privacy page.
- No third-party analytics or trackers on any surface. No durable trails of moderator in-event actions; the audit log is reserved for privileged admin/entitlement actions only (short defined lifetime; 12-month default, operator-configurable like every platform value).
- All moderator-authored text renders **inert** — never auto-linked, never clickable, no images (the M11 branding logo is the sole, entitlement-gated exception). The same inertness extends to every generated export file (user content escaped, never executable) and to display hygiene (direction-override characters stripped at intake).
- Surfaces beyond the public marketing pages are excluded from search-engine indexing **by default** (fail-closed noindex on every response; indexability only by deliberate allowlisting — capability URLs and event surfaces can never leak into an index by omission).

## VI. Committed events are sacred

- Lockdown, revocation, purges, deploys, and platform caution never break a running or scheduled event: **fail closed for creation, fail open for running events.**
- Sole termination paths: the moderator's own end action, an admin force-end, the emergency stop. Self-service account deletion requires ending a running event first.
- Rescheduling a committed event is stewardship, never a gated act. Releases must be deployable while events run (reconnect-safe by design; schema changes expand-first).
- The emergency stop is reversible quiescence and must be reversible **without a redeploy**; delete-all-data is a separate, honestly-labeled irreversible act. Every admin control shows a quantified consequence preview and defines its reverse action.

## VII. One entitlement seam

- Anything gated (features, counts, limits) is checked exclusively through a single capability interface in the service layer. Grants come from the free baseline, invite codes, admin action — and possibly billing later, which writes identical grants. **Gated code never knows where a grant came from.**
- The seam is anticipated architecturally from M1 even though the entitlement system ships in M8. Never-gated capabilities (core Q&A mechanics and all moderation/safety tools) are never routed through revocable grants.

## VIII. Admin access constraints

- `/admin` sits behind **one dedicated mandatory auth wall**, independent of moderator auth — **the wall is the admin auth**: the admin is not a product user, and product auth (magic links) plays no role in admin access. The control plane must never depend on the product's own email pipeline.
- Multiple wall mechanisms may exist (currently: password quick-start; platform access layer), but **exactly one is active at a time, selected by configuration with fixed precedence — the stronger mechanism always wins.**
- Whatever the active mechanism, three constraints are permanent: **emergency reset must always be possible via Cloudflare account access alone** (no lockout dead-end, no redeploy); the wall must **fail closed** if its enforcement layer is missing or misconfigured — including a fresh deploy with no wall configured, which yields setup instructions, never an open admin UI; admin identity is managed exclusively platform-side (secrets/allowlist — no in-product admin management).
- Destructive actions require step-up confirmation regardless of the wall.

## IX. Quality gates

- Strict typing; schema validation at every trust boundary (HTTP, WebSocket, storage, config) plus bounded input sizes at the edge; no unchecked escape hatches.
- Domain logic (state machine, auth, entitlements, purge) has unit tests covering every transition and invariant; **every data lifetime maps to exactly one enforcement job and one test proving deletion actually happens.**
- **No untested behavior ships:** a pull request that introduces or changes product behavior must include tests covering that behavior — feature code without tests does not merge. (Bugfixes start from a failing test; that rule is the same principle.)
- CI (lint, typecheck, test, build, secret scan) must be green to merge; the main branch is protected; deploys happen only via CI, never by hand.
- An unextracted user-facing string fails CI (see XI).

## X. Spec-driven change

- No feature code without spec → plan → tasks under `specs/`. Drift is corrected by amending artifacts (or converge), never by silently diverging code. Point-in-time external facts are re-verified at use time, never trusted from memory.

## XI. Accessible, phone-first, multilingual surfaces

- The participant experience is designed for phones in a dark event hall: readable, thumb-reachable, WCAG AA, graceful on flaky venue Wi-Fi — and device-agnostic ("participant surface", not "phone").
- Every user-facing string flows through the i18n layer from the first commit; adding a locale is a translation task, never a refactor. UI chrome is localized; user content never is.
- Motion explains state; nothing else moves. Failure copy is calm and honest, frames recovery, and never shows error codes or technical vocabulary on participant surfaces.

## XII. Fairness constraints on abuse handling

- Abuse defense must never punish a room for sharing a network: **mutating participant actions are never rate-limit-keyed by bare IP** (venue-NAT constraint, FR-5); entry paths stay zero-gate with room-scale ceilings.
- Abusive participants are the moderator's problem (skip-stub, hide board, primer, filters, limits); abusive moderators/events are the admin's. No in-product report button; no identity to ban on the participant side.
