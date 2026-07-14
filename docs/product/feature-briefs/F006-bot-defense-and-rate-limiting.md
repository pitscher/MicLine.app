# F006 — Bot defense & rate limiting

**Milestone:** M4 — Public-beta readiness · **Target version:** `v0.4.0` · **Implements:** SAFE-01, SAFE-02
**Depends on:** F002/F003/F005 (the guarded actions exist). **Manual prerequisites:** row 3 (Turnstile widget, both hostnames) + `TURNSTILE_SECRET_KEY` (row 4).

## User value

These are the beta window's actual defense (FR-1): with no event-count cap until M8 and no operator force-end until M6, rate limits + Turnstile + the kill-switch are what keep a public MicLine affordable and abuse-boring for a solo operator — without ever adding friction to watching or throttling a room that shares one venue NAT.

## In scope

- **Turnstile (SAFE-01):** invisible, on exactly three actions — auth request, event creation, first submission per participant session (re-runs after leave-and-rejoin per PRT-04); zero-gate entry stays CAPTCHA-free; server-side siteverify; documented test keys locally/CI.
- **Rate-limit matrix (SAFE-02):** the decided two-tier design and key matrix (tech-context §15) with **final numeric values decided in this spec** (and revisited with the OQ6 limit review): magic-link per-email + per-IP burst · event creation per-uid · **rescheduling** = burst brake + per-event daily cap (T8-5) · submission per-sid + per-event ceiling · entry/lookup paths per-IP at room-scale ceilings.
- **Venue-NAT constraint as tests:** a simulated room behind one IP must pass entry + submission flows unthrottled (FR-5, binding).
- **Per-DO soft throttle:** the EventRoom sheds excess load as calm retry states, never raw errors.

## Out of scope

Entitlement caps and lockdown (M8) · force-end/ban (M6) · emergency stop (M7) · WAF/Turnstile-mode escalation levers (documented in the M12 runbook, OPS-05).

## Dependencies & seams

All limits behind the one `ratelimit.ts` interface; every threshold is a platform-config value (F008), not a literal.
