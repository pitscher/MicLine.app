# F005 — Participant surface

**Milestone:** M3 — Open-line event runtime · **Target version:** `v0.3.0` · **Implements:** PRT-01, PRT-02, PRT-03, PRT-04, PRT-05, PRT-06, PRT-07, LINE-05, LINE-06
**Depends on:** F003.

## User value

Scan → see the board. Nothing between a participant and the line: no account, no name prompt to watch, no banner, no permission dialogs. Asking costs one text field; from then on the surface answers the only questions that matter — *where am I, and when am I up?* — and survives reloads and Wi-Fi drops without losing identity or place.

## In scope

- **Zero-gate join (PRT-01/02):** QR / `/e/⟨CODE⟩` / code box (a minimal code-entry page stands in until F007's landing) → live board view instantly; per-event anonymous identity cookie per tech-context §11.3.
- **Name modes (PRT-03/04):** localized funny nicknames (default) or required real name before **first submission** (unverified, locked); explicit "you'll appear as ⟨Name⟩" confirm; **leave event** = session reset (prior questions keep attribution; Turnstile re-run seam for M4).
- **Pre-join primer (PRT-05):** optional plain text; skippable vs required confirmation with the moderator's custom statement; **a gate, not a record** — ack lives only in the cookie.
- **Join-state screens (PRT-06):** lobby ("starts at ⟨time⟩"), event-full (concurrent-presence cap, no waitlist), ended, unknown/expired code.
- **The line experience (LINE-05/06, PRT-07):** full ordered line with Now speaking pinned; own-question chips (in line #N / answered / skipped; withdrawn disappears); position cues — sticky "You're #N" → "You're up soon" (approaching the top) → full-screen "You're up!"; withdraw (slot frees instantly); announcement banner with uniform expiry.
- **Save-state UI (T9-2 ③):** sending / saved as #N / failed-tap-to-retry — this surface owns that UI.

## Out of scope

Marketing landing + rescue lane (F007) · complete failure-state inventory + stale chip (F012) · pool zone, upvotes, declined/merged chips (M9) · re-openable ⓘ primer (M10) · push notifications (PRK-13, parked).

## Dependencies & seams

Position derivation is client-side from the broadcast line per visibility rules — no new server state. Phone-first, dark-hall-first, WCAG AA, device-agnostic wording ("participant surface"). All strings via Lingui (en+de). Checklist step mandatory.
