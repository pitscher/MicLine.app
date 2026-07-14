# F004 — Moderator console

**Milestone:** M3 — Open-line event runtime · **Target version:** `v0.3.0` · **Implements:** MOD-01
**Depends on:** F003 (consumes its WS contracts only).

## User value

Ground control on one phone-capable screen: the moderator runs a live event — sees the line and the live participant count, marks answered, skips with confidence (undo in reach), sends announcements, ends the event — without ever hunting through menus while a room watches.

## In scope

- One screen (MOD-01): header with title, share panel (code/URL/QR from F001), **live participant count**, end button; the line with answered/skip actions + undo toasts; announcement composer showing the expiry behavior; primer preview.
- `useEventSocket` reducer hook: opens the WS, reduces snapshot+deltas into state, treats server echo as authoritative, auto-reconnects with backoff into a fresh snapshot.
- Reconnecting state for the console (the full console failure-state set is completed by F012).

## Out of scope

Intake close, hide board, regenerate board link, follow-up flag (M5) · pool/review tray, drag-reorder, filters, search, bulk verbs, timestamps/per-submitter counts (M9) — **no dead scaffolding for them** · admin-ended/emergency notices (placeholders arrive with F012; behavior M6/M7).

## Dependencies & seams

Consumes ONLY `packages/shared` WS contracts; design tokens from `packages/ui` only; all strings via Lingui. Checklist step (P5d) mandatory: keyboard operability, reconnect banner, empty states, undo timing.
