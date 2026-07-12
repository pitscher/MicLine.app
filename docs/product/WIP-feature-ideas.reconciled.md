# MicLine.app — WIP Feature Ideas Reconciled for Product Interview

> **Purpose:** This document is an updated version of `WIP-feature-ideas.md`. It prepares a structured product interview by reconciling the WIP ideas with the current leading documents: `product-brief.md`, `feature-inventory.md`, and `decision-log.md`.
>
> **Authority rule:** The leading documents remain authoritative. This document does **not** redefine the product. It classifies WIP ideas as confirmed, adjusted, open for interview, parked, or rejected.
>
> **Do not merge directly:** Items marked `open-interview` or `adjusted-candidate` require explicit product-interview decisions before they can be incorporated into the leading documents.

---

## 1. Reconciliation Summary

The original WIP feature landscape was broader than the current product definition. The main corrections are:

| Area | Original WIP direction | Reconciled direction |
|---|---|---|
| Product container | `Presentation` containing one or more `sessions` | Use **event** only. One event = one room/session. No umbrella, presentation, or series concept. |
| Participant surface | `Question Line` as a broad participant page | Use **participant board / phone surface** and **the line**. The line is the queue; the board/phone surface displays and interacts with it. |
| Projector surface | `Question Board` | Use **the board** / **projector board**. It is read-only and deliberately minimal in M1. |
| M1 session start | Manual moderator start | Event auto-opens at required start time. Moderator can manually end. |
| M1 line behavior | Moderator can reorder, move up/down, reopen, move to later | M1 is **Open line / FIFO** only. Moderator can mark top item answered, skip any item, and undo briefly. Curated ordering arrives in M2. |
| Participant access | Passwords, company-domain verification, invite-based access | Preserve **zero-gate participation**: QR/URL/code leads directly to the board. Friction may guard submission, not presence. |
| Anonymity | Per-question anonymity toggle | Rejected by leading docs. Use event-level name modes: automatic nicknames or required real names before first submission. |
| Rich event content | Text, links, images in an event information panel | Current confirmed scope: M1 plain-text primer; M2 markdown-lite for primer/announcements with **no links**. Images are not currently confirmed. |
| Invite codes | Participant/session access codes | Invite codes are **moderator account entitlements / vouchers**, not participant access controls. |
| AI moderation | AI duplicate detection milestone | No AI milestone in the leading docs. M2 includes human-controlled duplicate merge in curated line; AI is parked unless explicitly revived later. |
| Milestones | M0–M6 product backlog | Use current execution order: **M1 → M2 → M4 → M5 → M3-if-ever**. No separate M0 or M6 product milestone. |

---

## 2. Canonical Product Frame

### 2.1 One-liner

MicLine is the live question line for every event: hosts collect, order, and moderate audience questions in real time, so everyone in the room knows who gets the mic next.

### 2.2 Product identity

MicLine is not primarily a generic Q&A board. Its identity is the **live mic line**:

- The line orders people who may speak.
- The current and next positions are visible.
- The product wedge is fairness, transparency, and low-friction participation.
- Moderators keep ground control without silently violating the line.

### 2.3 Pillars

| Priority | Pillar | Practical implication |
|---:|---|---|
| 1 | The line | Position stability and visible order are core. |
| 2 | Zero friction | Participants scan and see the event immediately. No account, app, cookie banner, or name prompt on entry. |
| 3 | Privacy | Minimal data, no trackers, short retention, honest privacy copy. |

### 2.4 Product boundaries

MicLine must not expand into:

- polls, quizzes, word clouds, reactions, or slide decks;
- livestream / webinar tooling for M1/M2;
- white-label, team-seat, or organization-account SaaS;
- classroom-suite positioning;
- public directories or umbrella conference structures.

---

## 3. Canonical Vocabulary Replacement Table

Use these terms in all future WIP discussions.

| Avoid / old WIP term | Use instead | Notes |
|---|---|---|
| Presentation | Event | One event = one room/session. |
| Session | Event | Do not introduce a separate session layer unless a future leading-doc decision changes this. |
| Question Line | The line / participant board / phone surface | `The line` is the queue. The participant phone surface is where participants view and interact. |
| Question Board | Board / projector board | Read-only big-screen route. |
| Creator | Moderator | The authenticated person creating and running the event. |
| Organizer | Moderator | Use moderator unless specifically discussing the future buyer/operator relationship. |
| Audience member | Participant | Participant has no account and no PII. |
| Alert message | Announcement / Mitteilung | One current live announcement in M1. |
| Session password | Not currently supported | Conflicts with zero-gate participation and the board as join funnel. |
| Invite / voucher | Invite code | Applies to moderator account entitlements, not participant entry. |
| Anonymous question | Event name mode | Anonymity is handled by automatic nicknames vs required real names, not per-question toggles. |

---

## 4. Status Labels Used in This Document

| Status | Meaning |
|---|---|
| `confirmed-covered` | Already covered by the leading docs. Do not re-decide unless the product interview needs implementation detail. |
| `adjusted-candidate` | The WIP idea is useful but must be reframed to fit the leading docs. |
| `open-interview` | Not currently confirmed. Needs an explicit product-interview decision. |
| `parked` | Deliberately out of current scope; may be revisited later. |
| `rejected-conflict` | Conflicts with confirmed product decisions. Do not carry forward unless the leading decision is intentionally reopened. |
| `technical-invariant` | Important, but should be captured as engineering/spec behavior rather than a product feature idea. |

---

# 5. Reconciled Feature-Idea Backlog

## 5.1 Confirmed or Already Covered

These ideas should not be treated as new feature proposals. They are already represented in the leading documents.

| WIP idea | Reconciled status | Leading-doc alignment | Interview action |
|---|---|---|---|
| Reliable participant continuity | `confirmed-covered` / `technical-invariant` | Covered by M1 realtime engine: Durable Object per event, WebSocket hibernation, snapshots/deltas, graceful reconnect. | Do not create separate product feature. Ensure engineering spec preserves participant identity, submitted questions, and position after reload/reconnect. |
| Safe question submission | `technical-invariant` | Covered by M1 realtime engine, state-machine invariants, content rules, and rate limits. | Capture as acceptance criteria: no silent loss, no duplicate submission from retries, clear save state. |
| Clear submission backpressure | `technical-invariant` / M5 topic | Aligns with graceful behavior on flaky venue Wi-Fi and future M5 observability/load testing. | Keep as engineering/operations acceptance criteria, not a standalone product module. |
| Visible event status | `confirmed-covered` | Scheduled, live, ended, full, unknown/expired are already part of join-state behavior. | Use current event-state language. Remove unconfirmed `paused` / `closing soon` states unless separately decided. |
| Question submission | `confirmed-covered` | M1 Open line: submitted question appends directly to public FIFO line. | No new decision needed. |
| Question length limit | `confirmed-covered` | 1–280 characters; trim, collapse whitespace, strip control characters. | No new decision needed unless changing the limit. |
| Basic participant question status | `confirmed-covered` | Own-question status chips: in line #N, answered, skipped; withdrawn disappears. | Preserve minimal chips. Avoid status taxonomy bloat. |
| Withdraw own question | `confirmed-covered` | Pulled into M1. Participant can withdraw their own not-yet-called question. | No new decision needed. |
| Moderator quick actions | `confirmed-covered`, but narrower | M1 actions are: mark top answered, skip any item, undo ~10 seconds. | Remove move up/down/reopen/move-later from M1. Reordering belongs to M2 curated line. |
| Announcement banner | `confirmed-covered` | M1 has one current announcement / `Mitteilung`, replacing the previous one, shown on board and participant phones. | Keep simple in M1. Rich formatting is M2. |
| Projector board | `confirmed-covered` | M1 board is read-only and minimal: QR/code, Now/Up-next, top of line, announcement area, fullscreen toggle. | Do not add M1 board modes/settings. Appearance controls belong to M4. |
| Post-event recap/export | `confirmed-covered` | M1 on-screen recap; M2 recap & export within the 3-day content retention window. | Interview should revisit whether export rank should move up because content is purged after 3 days. |
| Event stats / operational metrics | `confirmed-covered` | M1 anonymous aggregates; M2 admin metrics wall; M5 observability/cost model/load test. | Keep metrics split: product recap vs admin operations vs longevity. |
| Invite codes | `confirmed-covered`, but reframed | Invite codes are moderator entitlement vouchers for registration/event creation/capabilities/limits. | Do not use invite codes as participant access control unless a major product decision changes. |
| Duplicate handling | `confirmed-covered`, but non-AI | M2 curated line includes duplicate-merge. Upvotes advise curation; votes never reorder the line. | Keep human-controlled. Do not introduce AI unless explicitly revived later. |

---

## 5.2 Adjusted Candidate Ideas for the Product Interview

These ideas are potentially useful but need explicit product decisions before they can enter the leading docs.

### CAND-01 — Event Information Beyond the Primer

| Field | Value |
|---|---|
| Status | `adjusted-candidate` |
| Related original idea | Event Information Panel |
| Current leading-doc coverage | M1 plain-text pre-join primer; M2 markdown-lite primer/announcements with no links |
| Potential user value | Gives moderators a place for event rules, context, agenda notes, and audience instructions. |
| Risk / conflict | Links and images are not currently confirmed. Links were explicitly avoided for rich primer/announcements due to phishing risk. Images add storage, moderation, privacy, and abuse surface. |
| Suggested interview framing | Decide whether the primer should remain only a pre-join briefing or become an in-event collapsible information panel visible from the participant phone surface. |
| Likely milestone if accepted | M2 or later. M1 should remain plain text only. |

**Interview questions:**

1. Should event information be visible only before joining, or also inside the live participant phone surface?
2. Should it be a separate collapsible panel, or simply a richer primer?
3. Are links allowed at all? If yes, are they plain visible URLs, allowlisted domains, or moderator-trusted only?
4. Are images allowed? If yes, what are the file limits, retention rules, and abuse controls?
5. Should this appear on the projector board, or only on participant phones?

---

### CAND-02 — Multiple Active Questions per Participant

| Field | Value |
|---|---|
| Status | `open-interview` |
| Related original ideas | Configurable Open Question Limit, My Questions Overview, Repeated-participant Indicator |
| Current leading-doc coverage | Not explicitly confirmed. Participants can submit and withdraw questions; exact active-question limit per participant is not defined in the leading docs. |
| Potential user value | Lets participants submit multiple independent questions during long events without losing ideas. |
| Risk / conflict | Can undermine line fairness if one participant fills many positions. Creates UI complexity and moderation pressure. |
| Suggested interview framing | Decide whether MicLine should allow multiple active line items per participant and, if so, whether this is a fixed platform limit or per-event moderator setting. |
| Likely milestone if accepted | M2 or later, unless product interview explicitly pulls a small version into M1. |

**Interview questions:**

1. Is M1 implicitly one active question per participant, or are multiple active questions allowed?
2. If multiple are allowed, what is the default limit?
3. Should the moderator configure the limit per event?
4. Should the limit count skipped/answered/withdrawn questions, or only active waiting items?
5. Should repeated-participant indicators appear to moderators, participants, or nobody?

---

### CAND-03 — Request to Speak Without a Written Question

| Field | Value |
|---|---|
| Status | `open-interview` |
| Related original idea | Request to Speak |
| Current leading-doc coverage | The product identity is live mic line, but M1 mechanics describe submitted questions with 1–280 characters. No first-class empty `request to speak` item is confirmed. |
| Potential user value | Strongly matches the mic-line metaphor: a participant may want the microphone without prewriting a question. |
| Risk / conflict | Could make moderation harder and reduce clarity for the audience. It may also weaken the usefulness of the public line if entries do not communicate topic intent. |
| Suggested interview framing | Decide whether a line item must always contain written text, or whether `I want the mic` can be a distinct item type. |
| Likely milestone if accepted | M2 candidate, unless considered essential to the core live-mic identity. |

**Interview questions:**

1. Must every line item have question text?
2. Should `request to speak` be represented as a special short system-generated text, such as “Wants to speak”?
3. Does it count against the same participant active-question limit?
4. Should moderators be able to disable this per event?
5. Should it appear differently on the projector board?

---

### CAND-04 — Pause or Close Question Intake While Keeping the Event Running

| Field | Value |
|---|---|
| Status | `open-interview` |
| Related original ideas | Pause Session, Pause Question Intake, Close Remaining Questions, Countdown and Timed Intake Closure |
| Current leading-doc coverage | Not confirmed. M1 has scheduled/live/ended states and manual end. Review mode arrives in M2. Speaker timer is ranked M2. |
| Potential user value | Lets moderators stop new submissions near the end while continuing to process the existing line. Useful for events with limited time. |
| Risk / conflict | Adds state complexity and may violate the simplicity of M1. Automatic closure can surprise participants unless clearly communicated. |
| Suggested interview framing | Separate three concepts: event pause, intake closed, and speaker timer. Only the smallest useful concept should be considered. |
| Likely milestone if accepted | M2 or later. Avoid in M1 unless the product interview makes it critical. |

**Interview questions:**

1. Does MicLine need an `intake closed` state distinct from `event ended`?
2. Should closing intake be manual only, timer-based, or both?
3. What should participants see when intake is closed but the line continues?
4. Should the existing line remain visible and withdrawable?
5. Should intake closure be allowed before the event starts?

---

### CAND-05 — Moderator Projection Safety / Board Blackout

| Field | Value |
|---|---|
| Status | `open-interview` |
| Related original idea | Emergency Question Board Blackout |
| Current leading-doc coverage | Admin emergency stop exists in M2; M1 projector board is minimal. No moderator-level board blackout is confirmed. |
| Potential user value | Gives moderators a fast way to hide the public board if inappropriate content appears or if the live event needs a visual pause. |
| Risk / conflict | Could weaken public transparency if overused. Needs a clear distinction from admin emergency stop and event pause. |
| Suggested interview framing | Consider a local, moderator-controlled projector safety mode that affects only the board, not the participant phone surface or line state. |
| Likely milestone if accepted | M2/M4 candidate; not M1 unless considered essential for live-event safety. |

**Interview questions:**

1. Is skip + profanity filter enough for M1 public-board safety?
2. Should moderators be able to temporarily hide the projector board without ending the event?
3. Should participants still see the line on their phones during board blackout?
4. What exact neutral message should the board show?
5. Should board blackout be audit-logged?

---

### CAND-06 — Moderator Notes and Follow-up Flags

| Field | Value |
|---|---|
| Status | `open-interview` |
| Related original idea | Moderator-only Notes |
| Current leading-doc coverage | Not confirmed. M2 recap/export includes possible outcomes and follow-up value, but private notes are not in the feature inventory. |
| Potential user value | Helps moderators capture follow-up obligations, routing, or context while reviewing questions after the event. |
| Risk / conflict | Notes are event content and must obey the 3-day purge. If exported, they may include sensitive data. Adds hidden state to an otherwise transparent product. |
| Suggested interview framing | Decide whether a lightweight `follow-up required` flag is sufficient before introducing free-text private notes. |
| Likely milestone if accepted | M2 with recap/export, or later. |

**Interview questions:**

1. Is a boolean `follow-up required` enough?
2. Are private free-text notes necessary?
3. Should notes be exportable?
4. Should notes ever appear to participants? Default answer should be no.
5. How does note retention work under the 3-day purge rule?

---

### CAND-07 — Moderator Console Filters for M2 Modes

| Field | Value |
|---|---|
| Status | `adjusted-candidate` |
| Related original idea | Moderator Console Quick Filters |
| Current leading-doc coverage | M2 adds curated line, upvotes, review mode, duplicate merge, and co-moderators. These likely need better console controls, but exact filters are not specified. |
| Potential user value | Helps moderators handle larger events and review/curate pools efficiently. |
| Risk / conflict | Overbuilt filtering can make the console feel like enterprise software, conflicting with phone-first simplicity. |
| Suggested interview framing | Tie filters only to M2 modes and only to proven moderator jobs. |
| Likely milestone if accepted | M2, as part of curated/review-mode specification. |

**Candidate filters to evaluate:**

- status;
- reviewed / unreviewed;
- duplicate group;
- upvote count;
- skipped / answered;
- participant nickname/name;
- follow-up flag if accepted.

---

### CAND-08 — Fairness Indicators in Curated Line Mode

| Field | Value |
|---|---|
| Status | `adjusted-candidate` |
| Related original ideas | Repeated-participant Indicator, Fairness Warning for Reordering |
| Current leading-doc coverage | M2 curated line allows moderator selection and ordering. The line remains a fairness product, but no specific fairness warnings are confirmed. |
| Potential user value | Helps moderators preserve perceived fairness when curating or reordering. |
| Risk / conflict | Warnings can become naggy or imply moral judgment. The moderator must retain final control. |
| Suggested interview framing | Consider subtle moderator-only indicators, not blocking rules. |
| Likely milestone if accepted | M2/M4 candidate after curated-line semantics are settled. |

**Interview questions:**

1. Should original submission time always remain visible to moderators?
2. Should repeated upward movement be indicated?
3. Should multiple active items from the same participant be grouped or badged?
4. Should any fairness signal be visible to participants?
5. Should reorder history be kept only until purge or also in audit logs?

---

### CAND-09 — Participant Waiting-Time Estimate and “Next Soon” Cue

| Field | Value |
|---|---|
| Status | `open-interview` |
| Related original ideas | Participant Waiting-time Estimate, “You Are in the Next Three” Alert |
| Current leading-doc coverage | M1 already has `You're #N` and full-screen `You're up!`. No waiting-time estimate or next-three notification is confirmed. |
| Potential user value | Helps participants prepare before being called. |
| Risk / conflict | Estimates may be wrong and create disappointment. Browser notifications add permission friction and should not be part of zero-gate entry. |
| Suggested interview framing | Decide whether the existing position banner is enough, or whether a lightweight in-app `soon` cue is needed. |
| Likely milestone if accepted | M4 or later; possibly M2 if tied to speaker timer. |

**Interview questions:**

1. Is `You're #N` sufficient?
2. Should MicLine show `You're up soon` when the participant enters the next 3?
3. Should time estimates be avoided entirely?
4. Should browser notifications be allowed only as explicit opt-in after submission?
5. Does this add enough value to justify extra state and UI?

---

### CAND-10 — High-load Degraded Modes and Board Fallback

| Field | Value |
|---|---|
| Status | `adjusted-candidate` |
| Related original ideas | Degraded Question Line Mode, Read-only Question Board Fallback |
| Current leading-doc coverage | M5 includes observability, cost model, retention enforcement, load test, rebuild runbook, and compliance pack. Degraded-mode specifics are not confirmed. |
| Potential user value | Helps MicLine remain predictable under poor venue Wi-Fi or high-load conditions. |
| Risk / conflict | Advanced degraded modes can become complex and require operator attention, conflicting with solo-operator economics. |
| Suggested interview framing | Treat as a post-load-test M5 topic, not a pre-M1 feature. |
| Likely milestone if accepted | M5 after measured load-test data. |

**Interview questions:**

1. What does “fail open for running events” require in concrete UI states?
2. Which non-essential live updates may degrade first?
3. Is question submission always the highest-priority action?
4. What should the board show if live updates lag?
5. Which metrics must exist before degraded mode can be trusted?

---

## 5.3 Parked or Rejected WIP Ideas

These should not be carried forward as active feature proposals unless a leading decision is intentionally reopened.

| Original WIP idea | Status | Reason |
|---|---|---|
| Presentation with multiple sessions | `rejected-conflict` | Confirmed model is one event = one room/session. Parallel tracks are separate events. |
| Manual session start | `rejected-conflict` | Event auto-opens at required start time. Manual end exists. |
| Full session pause | `open-interview`, but not M1 | Not confirmed. If revived, split into intake closure, projector pause, and event end. |
| Custom Question Board password | `rejected-conflict` | Conflicts with zero-gate participation and the board as physical join funnel. |
| Verified company-domain participant access | `rejected-conflict` | Conflicts with zero-gate, no participant PII, and no participant accounts. |
| Single-use participant invite codes | `rejected-conflict` | Invite codes are moderator entitlement vouchers, not participant session-access controls. |
| Per-question anonymity | `rejected-conflict` | Rejected in favor of event-level name modes: automatic nicknames or required real names. |
| Panelist assignment | `parked` | Closest leading-doc analogue is topics/tags / “questions for presenter B,” which is parked for scale. |
| Focused Question Board mode in M1 | `rejected-conflict` for M1 | M1 board has one deliberately minimal layout. Appearance controls arrive in M4. |
| AI duplicate question detection | `parked` | M2 duplicate merge is human-controlled. No AI milestone exists in the leading docs. |
| Public duplicate indicators | `parked` | Not confirmed; could undermine line clarity and requires explicit session-level decision. |
| Browser notifications by default | `rejected-conflict` | Would add permission friction. If ever used, must be explicit opt-in after participant action. |
| Participant-facing recap page | `parked` | Already parked in leading docs due to retention minimalism. Export covers the need. |
| Topics/tags | `parked` | Parked until scale demands it. |
| Live reactions | `parked` | Not part of the line product; possible opt-in someday but no current milestone. |
| Waitlist for full events | `parked` | Leading docs specify clean full-event screen, no waitlist. |

---

# 6. Interview-Ready Product Questions

The structured product interview should focus on the following questions in order.

## 6.1 Must-clarify before M1/M2 specs

1. **Active question limit:** Is there one active question per participant, or can participants hold multiple positions in the line?
2. **Request to speak:** Must every line item contain written question text, or is a no-text `request to speak` item part of the live-mic identity?
3. **Intake closure:** Does MicLine need an `intake closed` state while the event stays live?
4. **Projection safety:** Is skip + profanity filter enough, or does the moderator need a board blackout/safety mode?
5. **Export priority:** Should recap/export move earlier within M2 because event content is purged after 3 days?

## 6.2 M2 mode-design questions

1. What exactly happens when a reviewed question is approved: append at approval time, keep original submission order, or let moderator place it?
2. What exactly is a duplicate merge in curated line mode?
3. Are rejected review-mode submissions completely invisible forever, or visible only to moderators until purge?
4. What console filters are necessary for curated/review mode?
5. Do fairness indicators belong in the curated-line console?

## 6.3 Event information questions

1. Is the existing primer enough, or should there be a live in-event information panel?
2. Should links stay forbidden even in richer M2 content?
3. Are images allowed despite the added abuse/storage/privacy surface?
4. Should event information be visible on the projector board or only on participant phones?
5. Does this feature belong to M2, M4, or later?

## 6.4 Operations and resilience questions

1. What minimum operational metrics are required before the first real public event?
2. Which M5 degraded-mode ideas should wait until after the load test?
3. What is the concrete UI copy for delayed saves, reconnects, and board lag?
4. How should the product communicate best-effort/no-SLA without scaring moderators?
5. Which failure states are participant-facing vs moderator-facing vs operator-only?

---

# 7. Recommended Clean Backlog View

This is the WIP backlog after reconciliation, expressed only as interview-prep candidates.

| Priority | Candidate | Status | Suggested milestone | Decision needed |
|---:|---|---|---|---|
| 1 | Multiple active questions + My Questions overview | `open-interview` | M2 | Decide active-question limit and fairness behavior. |
| 2 | Request to speak without written question | `open-interview` | M2 or M1 if core | Decide whether line item text is mandatory. |
| 3 | Event information beyond primer | `adjusted-candidate` | M2+ | Decide scope: primer-only vs live panel; links/images yes/no. |
| 4 | Intake closed while event continues | `open-interview` | M2+ | Decide state model and participant copy. |
| 5 | Moderator projection safety / board blackout | `open-interview` | M2/M4 | Decide whether public board needs a local safety mode. |
| 6 | Moderator notes / follow-up flags | `open-interview` | M2 | Decide if follow-up flag is enough or notes are needed. |
| 7 | Console filters for curated/review mode | `adjusted-candidate` | M2 | Decide minimum filters after M2 mode semantics. |
| 8 | Fairness indicators for curated line | `adjusted-candidate` | M2/M4 | Decide subtle moderator-only signals. |
| 9 | Waiting-time estimate / next-soon cue | `open-interview` | M4+ | Decide whether position banner is enough. |
| 10 | Degraded live modes / board fallback | `adjusted-candidate` | M5 | Decide after observability/load-test data. |

---

# 8. Updated Product Rules for WIP Ideas

Future WIP ideas should pass these filters before entering a product interview.

## 8.1 Line rules

- The line is the product.
- In M1, the line is FIFO and position stability is intentional.
- Votes, AI, admin tools, or entitlements must never silently reorder the line.
- Curated ordering is a visible M2 mode controlled by humans.

## 8.2 Participant rules

- Participants do not create accounts.
- Participants do not provide PII just to watch.
- Viewing must remain zero-gate.
- Friction may guard actions such as first submission, not presence.
- Browser permissions are opt-in only and never part of entry.

## 8.3 Moderator rules

- The moderator is the only authenticated human in M1.
- Moderator auth is email magic link only.
- The moderator must keep ground control without turning MicLine into enterprise admin software.
- M1 console stays phone-capable and minimal.

## 8.4 Board rules

- The M1 projector board is a read-only physical join funnel and public line display.
- M1 has one board layout.
- Board appearance controls are M4.
- Any board blackout/safety mode must be explicitly decided and clearly separated from admin emergency stop.

## 8.5 Privacy and retention rules

- Event content is purged 3 days after event end.
- Identifying content must not survive deletion.
- Anonymous aggregates may survive only if unlinkable.
- Moderator notes, images, links, exports, and event information all increase retention/privacy obligations.

## 8.6 Scope rules

- No polls, quizzes, word clouds, slide decks, reactions, or livestream ingestion.
- No company-domain participant gating for current scope.
- No white-label/team/seat/org model.
- No participant access-control model unless zero-gate is intentionally reopened.

---

# 9. Suggested Updates to the Original WIP File

If `WIP-feature-ideas.md` remains a living artifact, apply these edits:

1. Rename the file to `WIP-feature-ideas.reconciled.md` or add `Status: superseded by leading docs` to the top of the old file.
2. Replace `Presentation` and `Session` terminology with `Event`.
3. Remove the M0 and M6 milestone structure.
4. Reclassify reliability items as M1 technical invariants or M5 longevity topics.
5. Replace manual start with scheduled auto-open and manual end.
6. Remove M1 reordering actions; move reordering to M2 curated line.
7. Remove board password, company-domain access, and participant invite-code access ideas.
8. Remove per-question anonymity and use event-level name modes.
9. Convert event information, request-to-speak, multiple active questions, intake closure, board blackout, and moderator notes into explicit interview questions.
10. Keep export/recap, announcements, withdraw own question, and operational metrics, but align them with their confirmed milestones.

---

# 10. Final Interview Preparation Notes

The strongest unresolved product tension is whether MicLine remains a very strict FIFO question line, or whether it should support richer live-event behaviors such as multiple active questions, request-to-speak items, intake closure, and board safety controls.

The safest interview posture is:

1. Protect the confirmed product identity first.
2. Treat every WIP feature as guilty until it proves it improves the line.
3. Avoid participant friction unless it protects an action.
4. Avoid M1 expansion unless the feature is essential for a real first event.
5. Prefer M2 candidates that strengthen moderation without weakening transparency.
