# F011 — Recap & account self-service

**Milestone:** M4 — Public-beta readiness · **Target version:** `v0.4.0` · **Implements:** EVT-05, AUTH-03, AUTH-04
**Depends on:** F010 (purge machinery + countdown), F002 (accounts).

## User value

Honest closure on both sides of an event: the moderator sees what happened and exactly when the content dies (no surprises on day 3); participants get a polite goodbye with a path onward. And from the first public release, a moderator can erase themselves completely in one self-sufficient flow — GDPR Art. 17 as product, not paperwork.

## In scope

- **On-screen recap (EVT-05):** post-end stats recap with the explicit content-deletion countdown; stats (survive) **visually separated** from content (dies); copy states that full export ships next (M5); participants get the polite end screen with "join another event".
- **Self-service account deletion (AUTH-03):** a running event must be ended first (one tap away); scheduled events listed in a **quantified consequence preview**; loading state → explicit "all your data has been purged"; **no confirmation email — the UI is the confirmation**; deletion honesty per the interview (aggregates survive unlinkably; the ≤30-day infrastructure-recovery sentence).
- **Inactivity auto-purge (AUTH-04):** 1 month, no warning email; carve-outs: future event scheduled (the wallet carve-out activates with M8); re-registration hint on the auth screen.
- **My-events extension (MOD-03):** past-until-purge + deletion-countdown states.

## Out of scope

Export files (M5 — the recap says so) · participant-facing recap (PRK-03, parked) · admin-side deletion/force-end (M6) · email-change flow (PRK-14, parked).

## Dependencies & seams

Deletion semantics per T9-5/T8b-2 are binding. Ending-first is a precondition, not a warning. Aggregation-at-deletion reuses F010's pattern.
