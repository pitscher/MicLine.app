# F012 — Failure states, feedback & locale completion

**Milestone:** M4 — Public-beta readiness · **Target version:** `v0.4.0` · **Implements:** PRT-08, MOD-04, I18N-02
**Depends on:** every M3 surface (F003–F005, F009), F002 (email registry for E-2).

## User value

Venue Wi-Fi dies, events fill up, links expire — and MicLine never looks broken: every failure resolves into a calm, honest state that frames recovery. German and English become genuinely production-complete on every surface, and the moderator gets a quiet channel back to the operator.

## In scope

- **Canonical failure-state inventory (PRT-08 — the list is CLOSED, T9-3):** delayed save · failed save with retry · participant "reconnecting — the line continues" banner · **stale-board chip** (the unattended board self-describes lag) · event full · unknown/expired code. Plus the **state-takeover rule** (races resolve into the new canonical state — ended/force-ended/emergency/intake-closed/full — with calm feedback, never raw errors) and the **moderator-console set**: reconnecting, action-failed-retry, admin-ended notice *placeholder*, emergency notice *placeholder* (behaviors arrive M6/M7).
- **Tone rule:** calm, honest, no error codes or technical vocabulary on participant surfaces; copy per locale; device-agnostic wording.
- **Feedback funnel (MOD-04):** optional post-event feedback box, **moderator-only surface** (FR-4), delivered as **E-2** through the template registry to `FEEDBACK_EMAIL` (Wrangler secret; the M6 admin setting takes over later).
- **Surface locale derivation (I18N-02):** the full four-surface table (console = device default, adjustable anytime, persisted on account · event language = device default at creation, adjustable before start only · participant surface = participant device default · board = event language, per-viewer until event end · emails = console language) implemented against the decided **explicit-override-only localStorage precedence rule**; the named edges (register-variant direction, dual-surface viewer, pre-start event-language change) are resolved in this spec. Production-ready `en`+`de` across every surface.

## Out of scope

Degraded modes (M12-only, post-load-test) · admin-ended/emergency behaviors (M6/M7) · `de (formal)` catalog (M10 / I18N-01) · new failure states (the list is closed).

## Dependencies & seams

E-2 goes through the registry — no ad-hoc send. The stale-board chip completes F009's seam; save states shipped in F005 are members of this inventory, not duplicates. Feedback submissions increment their AF-3 ops counter (tech-context §16).
