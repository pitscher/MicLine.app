# F003 — EventRoom realtime + the open line

**Milestone:** M3 — Open-line event runtime · **Target version:** `v0.3.0` · **Implements:** LINE-01, LINE-02, LINE-03, LINE-04, LINE-07, LINE-08, MOD-02, SAFE-03, SAFE-04
**Depends on:** F001, F002. **The heart** — Opus-class, max plan effort, two-phase implement (Phase A: schema + state machine + services + unit tests, no sockets · gate · Phase B: WS transport, snapshot/deltas, scripted two-client integration test). Gate-2 note: SAFE-03/04 confirmed to ride inside this spec.

## User value

The product's category claim becomes running truth: a fair, transparent line where **order = arrival, your position only ever improves**, everyone sees the same state live, and flaky venue Wi-Fi never costs anyone their place. The moderator's ground control (answered, skip, undo, announcements) works in seconds without ever surprising the room.

## In scope

- **The open-line state machine (LINE-01/02/03/04):** submit → appended to the public FIFO line; Now speaking = head; answered auto-pulls next; **skip with stub semantics** (text off all public surfaces instantly; attribution + "skipped" stub in the participant line; board never shows skipped; owner keeps a chip); ~10 s single-level undo for answered and skip; full statuses/invariants/visibility per tech-context §8 — **two-mode-shaped schema, zero curated machinery**.
- **One active waiting item per participant (LINE-08):** session-scoped, fixed platform rule; answered/skipped/withdrawn frees the slot; no fingerprinting or IP binding, ever (T1-2).
- **Realtime engine (LINE-07):** EventRoom DO per event, hibernation WS, role/pid tags, snapshot+deltas, graceful reconnect; the **T9-2 acceptance criteria appear verbatim in the spec** (no silent loss · duplicate-free retries · explicit save states · automatic recovery).
- **Presence & event-full (FR-2):** concurrent-presence counting, reconnect grace (config value, direction ~60 s — final number in this spec), cap admission, live count.
- **Announcements (MOD-02):** one current, replaces previous, uniform auto-expiry from platform config via the DO alarm, clear-early, re-send restarts.
- **Content rules (SAFE-03) + profanity filter (SAFE-04):** enforcement at the submission boundary (trim, 1–280 + per-event bounds, whitespace collapse, control-char strip — explicitly incl. bidi/direction-override characters, AF-1; optional per-event wordlist filter, default off — block-vs-mask semantics decided here per FR-5); moderator-authored text renders inert.
- **Ops counters (AF-3):** writes the event-runtime counters it owns — started / ended manually / auto-ended, event-full rejections, peak concurrent presence (tech-context §16).
- **Alarm multiplexer:** announcement expiry, grace expiries, auto-end broadcast (self-purge slot added by F010).
- **Tests:** every transition, the one-Now and one-active-item invariants, skip-stub visibility, undo windows, announcement expiry, presence admission — **the ≥80 % service/DO coverage gate starts here**.

## Out of scope

Pool, votes, review, declines, merge, reorder machinery (M9 — schema-ready only) · all UI (F004/F005/F009) · purge jobs (F010) · rate-limit numbers (F006) · nickname generation language + uniqueness details (this spec decides them per FR-5 if F005 doesn't).

## Dependencies & seams

WS contracts are zod schemas in `packages/shared`, consumed verbatim by DO and clients. Nothing stateful may exist outside the DO (architecture rule 2). Ordering schema must accept M9's fractional `order_key` without rewrite.
