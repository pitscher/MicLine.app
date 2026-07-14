# F009 — Projector board (minimal)

**Milestone:** M3 — Open-line event runtime · **Target version:** `v0.3.0` · **Implements:** BRD-01
**Depends on:** F003 (contracts), F001 (the capability token it sits behind).

## User value

The wall shows the room who's next and how to join — huge, readable from the back row, impossible to stumble into via a guessable URL. The join funnel (QR + code) never leaves the screen, so latecomers onboard themselves in a dark hall.

## In scope

- Read-only big-screen route behind the **capability URL minted in F001** (`/b/⟨uuid⟩` — never exposes event ID or join code; unknown token = the same calm screen as unknown codes).
- Join QR + code always visible · Now / Up-next huge · top of the line · announcement area (with the subtle draining self-dismissal cue) · fullscreen toggle.
- **One layout, no settings.** Never shows skipped items, timestamps, or vote counts. Board language = event language (per-viewer language choice arrives with I18N-02 in M4).
- High-contrast, dark-first, projector→TV legible.

## Out of scope

Hide board + link regeneration (M5) · per-viewer language + stale-board self-description chip (M4 / F012 — leave the seam) · pool zone + collapse (M9) · appearance controls + sound cue + responsive overhaul (M11).

## Dependencies & seams

Same `packages/shared` WS contracts as every surface. Checklist step mandatory: projector legibility, fullscreen behavior, reconnect.
