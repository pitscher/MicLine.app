# F002 — Moderator auth (magic links)

**Milestone:** M2 — Moderator identity and event setup · **Target version:** `v0.2.0` · **Implements:** AUTH-01, AUTH-02
**Depends on:** F001 (replaces its auth stub). **Manual prerequisites:** rows 2 + 4 (email domain onboarded, `SESSION_SECRET` set) before the verify step. Model note from file 06: Opus-class for implement.

## User value

One email, one click, ~30 days signed in, unlimited events — no passwords, no profiles, nothing to breach beyond an address. Anti-enumeration and silent-ban behavior protect users and operator alike without a single user-visible security ceremony.

## In scope

- **Magic-link flow (AUTH-01):** exactly tech-context §11.1 — always-200 request with uniform timing, hashed single-use tokens in D1 with atomic consume, 15-minute validity, calm expired-link screen; **E-1 as the email-template registry's first real template** (Lingui, moderator's console language); banned/soft-deleted addresses silently receive nothing (ban tooling itself is M6 — the seam is honored here).
- **Sessions:** stateless HMAC cookie per tech-context §11.2; `requireUser()`/`requireAdmin()`; logout; per-request user-row check so bans/deletions bite immediately.
- **Minimal account (AUTH-02):** email + optional display name + country (request geolocation, aggregates only) + persisted console language — **nothing else**; registration = first successful verify.
- **Seams left, not wired:** Turnstile on the request (enforced in F006), M8 lockdown gate on registration.

## Out of scope

Self-service deletion + inactivity purge (F011) · Turnstile enforcement (F006) · admin wall (M6 — Access, tech-context §11.4) · email-change flow (PRK-14, parked).

## Definition of done (manual verify)

A real magic-link mail arrives (deploy preview or `remote: true`), SPF/DKIM/DMARC pass in the received headers, login round-trips on DEV, and the local-dev path (simulated send → read link from the local file/log) is documented in the repo.
