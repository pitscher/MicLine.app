# F001 — Event creation & scheduling

**Milestone:** M2 — Moderator identity and event setup · **Target version:** `v0.2.0` · **Implements:** EVT-01, EVT-02, EVT-03, EVT-04, MOD-03
**Depends on:** F008 (config values, schema baseline). Auth is a stub until F002 replaces it — the fake session exists only under `ENVIRONMENT=local`, never on deployed envs (tech-context §3, S1 rule).

## User value

A moderator creates a real, scheduled event in under a minute and immediately holds everything needed to put it on printed material or a slide **up to 14 days ahead** (FR-3): the 6-char join code, the `/e/⟨CODE⟩` URL, the QR, and the board capability URL — while venue chaos stays survivable through anchored ±24 h rescheduling.

## In scope

- **Creation form (EVT-01):** required title, start & end date+time (create-ahead window from platform config); optional private description and primer (+ acknowledgment mode + custom confirmation statement); settings: event language, name mode (nicknames default / real names), profanity filter toggle, per-event min/max question length within platform 1–280.
- **Scheduling lifecycle (EVT-02):** Scheduled → auto-opens at start → ended manually (console arrives M3) or auto-end at end+24 h; start/end each adjustable ±24 h around **original** values, unlimited count within the window, built rate-limitable (matrix numbers land in F006); bigger moves = delete + recreate (new join code, stated in UI).
- **Settings freeze (EVT-03):** immutable once live; scheduling fields keep their window; live controls are not settings.
- **Join codes & share panel (EVT-04):** Crockford-base32 code with collision retry; KV code map with D1 fallback+backfill; `/e/⟨CODE⟩` + QR in the console share panel (the participant runtime behind the URL is M3).
- **Board capability token:** minted here per tech-context §12 (UUID; the board page is F009, regeneration M5).
- **My-events list (MOD-03):** scheduled / active / past, sorted by date (purge-countdown states arrive M4, admin-removed state M6).

## Out of scope

Real authentication (F002) · join/participant runtime (M3) · recap, countdown states, deletion (M4) · board page (F009) · link regeneration and live-safety controls (M5) · duplicate-event (M10).

## Dependencies & seams

Lifecycle timing rides the decided persistence architecture (auto-open/auto-end via DO alarm + cron backstop — tech-context §10). Every limit/value reads from F008's config, never a literal.
