> **MicLine Build Guide — Part 4 of 10** · [◀ GitHub operations: environments, releases, lifecycle](03-github-operations.md) · [⌂ Index](README.md) · [The feature loop ▶](05-feature-loop.md)

**In this part:** The product-definition prompts turn "MicLine, roughly" into a product brief, tech context, constitution, and epic briefs — with gates 1–4 as your steering wheel. P1 and P1.1 (plus the milestone revamp and the FR final review) are executed history; P2–P4 are what runs next.
**Prerequisites:** File 02 complete · file 03 read · ideally the Fable 5 window still open (file 01). · **Your time:** ~2–4 h of your attention spread over 1–3 days (the P1 interview dominates). *(your active attention; Claude runtime and limit pauses excluded)*

> **Numbering disambiguation (read once, saves confusion forever):** this guide numbers its *prompts* P1–P5. The product artifacts number their *provenance phases* P1/P2/P3 — there, **P1** = the first product interview (prompt P1, 2026-07-06), **P2** = the follow-up interview (prompt P1.1, 2026-07-12), **P3** = the milestone restructuring (2026-07-12) that produced the approved **M1–M13** lineup. The prompt blocks below are preserved **verbatim as executed** — milestone references *inside* the P1/P1.1 blocks reflect the pre-P3 numbering of their day (e.g. "M2 stand-out features") and are historical, exactly like the annotated early entries of the decision log. Everything *outside* those blocks uses the current M1–M13 lineup.

## §3 Phase 1 — Product brainstorm (the interview you asked for)

Before the prompts, the mental model you asked for — **how each building block goes from idea to running code**. Nothing is built ad hoc; every block passes the same three stations: *envisioned* (the P1 interview), *pinned* (P2 tech context, P3 constitution, P4 briefs), *built* (the file-05 loop on a numbered feature):

| Building block | Envisioned in | Pinned in | Built as |
|---|---|---|---|
| Landing page / marketing | P1 topics B + F | F007 brief | F007 loop, M4 (made *beautiful* in M11) |
| Moderator console & participant surfaces | P1 topic B | F003–F005/F009 briefs + `packages/ui` tokens | M3 loops (functional first — the deliberate beauty pass is M11, so you never fight design battles twice) |
| Backend & realtime core | P1 topics B + E | tech-context + constitution Art. IV + F001–F003 briefs | M2–M3 loops |
| Admin interface | P1 topic D | M6/M7 epics ("Demands on M1–M4" reach back into the early schema) | M6/M7 loops |
| Vouchers / entitlements | P1 topic E | constitution Art. VI + M8 epic | M8 loops (seam anticipated from M1) |
| Locales / i18n | P1 topic H | tech-context + constitution Art. IX | F000 plumbing (M1), then every UI feature; completion M4 |

Goal of this phase: turn "MicLine + some milestone ideas" into a complete, *your-words* product definition **before** any Spec Kit artifact exists. Claude interviews you; you answer; Claude writes it down. Budget: one or two sessions, 60–120 min of your time. This is Fable-window work.

> Session hygiene for a long interview: if `/context` shows the window filling up (or answers get vague), say `Write everything captured so far into the three output files now`, `/clear`, then resume with: `Read docs/product/*.md and continue the interview where the OPEN QUESTIONS section left off.`

```text
PROMPT P1 — product definition interview
[MODEL: Fable 5] [EFFORT: max] [SESSION: fresh]

You are the product strategist for MicLine.app — "the live question line for
every event": hosts collect, order, and moderate audience questions in real
time so everyone knows who gets the mic next. If docs/inputs/tech-decisions.md
and/or docs/inputs/legacy-spec.md exist, read them for background — but treat
them as drafts to be challenged, not as decided. If neither exists, start from
this one-line pitch alone.

We will now define the product through a structured interview. Rules:

1. Interview me ONE topic at a time, max 5 questions per batch, numbered so I
   can answer by number. Wait for my answers before the next batch.
2. Be critical: when my idea is vague, has hidden complexity, conflicts with an
   earlier answer, or adds maintenance burden for a solo operator, say so and
   propose alternatives — do not just transcribe.
3. Where I say "you decide", make the call, mark it ASSUMED, and move on.
4. Maintain a running OPEN QUESTIONS list for anything I defer.

Topics to cover, in order:
  A. Audience & positioning — event types (conference Q&A? meetups? classrooms?
     livestreams?), who pays eventually, what makes a moderator choose MicLine
     over Slido/raised hands.
  B. M1 core walkthrough — for each surface, walk me screen by screen and
     challenge scope: (1) landing page, (2) moderator console, (3) audience
     board, (4) join flow. Confirm or amend every per-event setting (moderation mode
     open/pre-moderation, anonymity, upvotes on/off, profanity filter, pool
     sort — see tech-decisions.md if present).
  C. M2 stand-out features — I have NOT defined these yet. Propose 10–15
     candidate features that would differentiate MicLine (draw on the live-Q&A
     competitor landscape), interview me on each (keep/kill/change), and force-rank
     the keepers.
  D. Admin interface (owner = me) — I must administer the SaaS with zero code /
     dashboard / DB access. Interview me on scope: user & event oversight,
     abuse handling, metrics, feature flags, announcements, data deletion.
  E. Voucher/entitlement system — my sketch: I generate voucher codes in the
     admin UI; each code grants a defined capability set (e.g. "all features,
     one event" or "N events" or "multi-use code"). Interview me until the
     semantics are exact: what is gated (features? event count? attendee count?
     event duration?), redemption rules, expiry, revocation, what a moderator
     without any voucher can do (the free baseline), and how a future billing
     plan would map onto the same entitlement grants.
  F. UI overhaul intent (M4) — capture design direction only (references,
     tone, brand adjectives), no design work.
  G. Operations (M5 seed) — my tolerance for cost, on-call, data retention,
     GDPR expectations as a Germany-based operator.
  H. Locales & tone — MicLine ships multilingual from day one (the plumbing
     lands in F000). Which locales at launch (my default proposal: en + de)?
     How is a locale chosen — browser Accept-Language with a manual switcher,
     and/or a per-event moderator setting for the participant surfaces? Tone
     of voice per surface and per locale.

When all topics are done, write:
  - docs/product/product-brief.md   (positioning, personas, surfaces, principles)
  - docs/product/feature-inventory.md   (EVERY feature, one line each, fields:
    id, name, one-sentence description, milestone, status: confirmed|assumed|open)
  - docs/product/decision-log.md   (each decision + why + alternatives rejected;
    ASSUMED items and OPEN QUESTIONS flagged)
Then print the feature-inventory table in chat for my review.
```

### Review of created product artifacts and refinement

The artifacts `product-brief.md`, `feature-inventory.md` and `decision-log.md` were created after the first product interview, I reviewed them. I also came up with some additional feature ideas and used GPT 5.5 with high effort to refine them. Afterwards, I instructed GPT to reconcile them based on the already created product interview artifacts. The result of this exercise is the document `WIP-feature-ideas.reconciled.md` (now archived at `docs/inputs/WIP-feature-ideas.reconciled.md` — the prompt below cites its original `docs/product/` path).

The follow up product interview prompt (below) was also created by GPT and will now be used with an empty Fable 5 chat to refine the already created product artifacts before the tech interview starts.

```text
PROMPT P1.1 — follow up product definition interview
[MODEL: Fable 5] [EFFORT: max] [SESSION: fresh]

You are the senior SaaS product strategist for **MicLine.app**.

MicLine is the live question line for every event: hosts collect, order, and moderate audience questions in real time, so everyone in the room knows who gets the mic next.

This is the **final product interview before the technical interview begins**. After this interview, the product artifacts will be used as the product foundation for the tech interview, specification work, and implementation planning with GitHub Spec Kit. Product decisions that affect scope, UX, privacy, moderation, operations, milestones, or future implementation must be discussed now, not silently deferred.

---

## 1. Required input files

Before asking questions, read these files carefully:

1. `docs/product/product-brief.md`
2. `docs/product/feature-inventory.md`
3. `docs/product/decision-log.md`
4. `docs/product/WIP-feature-ideas.reconciled.md`

The first three files are the **leading documents** created during the first product interview. They are authoritative unless the operator explicitly reopens or changes a decision during this interview.

`WIP-feature-ideas.reconciled.md` is not authoritative. It is an interview-preparation document. Use it to identify unresolved product decisions, adjusted candidates, conflicts, parked ideas, and areas that need final clarification.

If any of the four files are missing or unreadable, stop and ask me to attach them before continuing. Do not rely on memory or assumptions from another chat.

---

## 2. Authority and conflict rules

Use this hierarchy:

1. Explicit decisions I make during this P1.1 interview.
2. Existing leading documents: `product-brief.md`, `feature-inventory.md`, `decision-log.md`.
3. `WIP-feature-ideas.reconciled.md` as structured input only.
4. Your product judgment, only when I say “you decide” or when a small decision is required to keep the interview moving.

When the WIP document conflicts with the leading documents, preserve the leading-document decision unless I explicitly choose to reopen it.

When I propose something that conflicts with MicLine’s product principles, challenge it clearly. Do not merely transcribe my idea.

Important confirmed principles to protect unless explicitly reopened:

* MicLine’s identity is the **live mic line**, not a generic Q&A board.
* One event = one room/session. No `presentation`, `session`, umbrella, series, team, seat, or organization model in current scope.
* Participant viewing is **zero-gate**: QR/URL/code leads directly to the event board/phone surface. No account, no name prompt, no cookie banner, no participant PII, no CAPTCHA on entry.
* Friction may guard participant actions such as first submission, not presence.
* Participants have no accounts.
* Moderators are the only authenticated humans in M1.
* M1 line mode is **Open line / FIFO**: submitted questions append to the public line; position stability is a feature.
* M1 moderator actions are intentionally narrow: mark top item answered, skip any item transparently, and undo briefly.
* Curated ordering, review mode, upvotes, duplicate merge, and co-moderators belong to M2.
* Votes, AI, admin tools, entitlements, or hidden logic must never silently reorder the line.
* Per-question anonymity was rejected in favor of event-level name modes: automatic nicknames or required real names before first submission.
* Invite codes are moderator entitlement vouchers, not participant access-control codes.
* Event content is purged 3 days after event end; anonymous aggregates may survive only if unlinkable.
* No polls, quizzes, word clouds, reactions, slide decks, livestream ingestion, white-label, teams/seats/org accounts, public directory, waitlist, or classroom-suite positioning in current scope.
* Open source is not a SaaS marketing pillar unless explicitly changed.

---

## 3. Interview behavior

Run a structured interview, not a one-shot rewrite.

Rules:

1. Interview me **one topic at a time**.
2. Ask **maximum 5 questions per batch**, numbered so I can answer by number.
3. Wait for my answers before moving to the next batch.
4. Be critical. Surface hidden complexity, product conflicts, privacy risks, solo-operator burden, and UX tradeoffs.
5. When I say “you decide,” make the call, mark it `ASSUMED`, and move on.
6. If I defer a decision, push once for a product-level default. If I still defer, add it to an `OPEN QUESTIONS` list with a clear owner and consequence.
7. Keep a running list of decisions, assumptions, superseded decisions, parked ideas, and open questions.
8. Prefer final decisions over open questions. This is the last product interview before tech work.
9. Do not design the technical architecture. However, do ask product-level questions that affect implementation scope, data lifecycle, moderation semantics, failure states, or Spec Kit readiness.
10. Do not invent features or claims not grounded in the attached documents or my answers.

After each topic, briefly summarize:

* confirmed decisions;
* assumptions you made;
* open questions;
* any feature-inventory changes implied.

Then proceed to the next topic.

---

## 4. Terminology to use

Use the current MicLine vocabulary consistently.

Preferred terms:

* `event`
* `moderator`
* `participant`
* `the line`
* `participant board` or `participant phone surface`
* `projector board` or `board`
* `announcement` / German UI term `Mitteilung`
* `pre-join primer`
* `invite code` for moderator entitlements only

Avoid unless explicitly discussing rejected/superseded concepts:

* `presentation`
* `session`
* `question board` as a generic product container
* `session password`
* participant accounts
* participant invite codes
* company-domain participant access
* per-question anonymity

If a final artifact contains old terms, it must be because the artifact is documenting a rejected or superseded idea.

---

## 5. Required interview topics and order

### Topic 0 — Reconciliation pass and interview framing

First, silently reconcile the four input files. Then give me a concise agenda for the interview and start with Topic 1. Do not ask me to restate the product.

Identify the WIP items that genuinely need product decisions and avoid reopening everything from P1.

---

### Topic 1 — M1 scope locks and line semantics

Resolve final M1 product behavior, especially where the WIP document surfaced gaps.

Cover:

* whether M1 allows exactly one active question per participant or multiple;
* whether a participant can create a line item without written question text, i.e. `request to speak`;
* whether written question text is always required under the 1–280 character rule;
* how participant withdrawal works if multiple line items or request-to-speak items exist;
* whether any `intake closed` state exists in M1, or whether M1 only has scheduled/live/ended/full/unknown states.

Be strict about M1 simplicity. Only pull an idea into M1 if it is essential for a real first event.

---

### Topic 2 — Event information, primer, announcements, and content richness

Resolve what moderators can communicate to participants before and during an event.

Cover:

* whether the current plain-text pre-join primer is enough;
* whether there should be an in-event collapsible information panel on the participant phone surface;
* whether rich M2 primer/announcement content remains markdown-lite with no links;
* whether links are ever allowed, and if so under what constraints;
* whether images are allowed at all, given storage, abuse, moderation, privacy, and 3-day purge implications;
* whether event information ever appears on the projector board or only on participant phones.

Challenge anything that weakens zero-gate participation, privacy minimalism, or solo-operator maintainability.

---

### Topic 3 — Projection safety, board behavior, and live-event control

Resolve whether moderators need safety controls beyond skip/profanity filtering.

Cover:

* whether M1 skip + optional profanity filter are enough for public-board safety;
* whether a moderator-level board blackout/safety mode exists;
* if board blackout exists, whether it affects only the projector board or also participant phones;
* neutral copy shown during blackout;
* whether board blackout is audit-logged;
* whether this is M2, M4, or rejected.

Keep this clearly separate from the M2 admin emergency stop, which ends all running events and blocks creation.

---

### Topic 4 — M2 line modes, review mode, duplicates, and upvotes

Finalize M2 semantics so the technical interview has no ambiguity.

Cover:

* curated line mode: pool → moderator selects/orders into the line;
* whether approved review-mode submissions append at approval time, preserve original submission order, or are manually placed;
* whether rejected review-mode submissions are invisible forever to participants and retained only until purge;
* what duplicate merge means in product terms;
* whether duplicate merge changes participant-visible line items, vote counts, attribution, or exports;
* how upvotes behave in pool mode and confirm again that they advise curation but never reorder the line;
* whether co-moderators are required for review mode to be usable at scale.

Avoid technical implementation details, but make product semantics exact.

---

### Topic 5 — Moderator console efficiency, notes, follow-up, and filters

Decide the minimum moderator tooling needed for M2 without turning MicLine into enterprise software.

Cover:

* whether a simple `follow-up required` flag is enough;
* whether private moderator notes exist;
* whether notes are exportable;
* retention of notes under the 3-day event-content purge;
* minimum useful filters for curated/review mode;
* whether filters differ between open line and curated/review modes.

Protect transparency: private moderator tooling must not undermine the participant-facing fairness promise.

---

### Topic 6 — Fairness indicators and participant preparation cues

Resolve whether MicLine needs additional fairness/preparation features beyond the confirmed position banner and `You're up!` cue.

Cover:

* repeated-participant indicators;
* original submission time visibility;
* reorder history in curated mode;
* whether fairness warnings are moderator-only, participant-visible, or rejected;
* whether `You're up soon` / next-three cues are needed;
* whether waiting-time estimates are rejected, parked, or accepted;
* whether browser notifications are allowed only as explicit opt-in after submission, or rejected entirely.

Challenge estimates and browser permissions because they can create trust and friction problems.

---

### Topic 7 — Recap, export, data lifecycle, and post-event value

Finalize the post-event product promise in light of the 3-day purge.

Cover:

* whether M2 recap/export should move earlier or higher in priority;
* exact export formats: CSV, JSON, both, or later;
* export contents and filters;
* whether moderator notes/follow-up flags are exportable;
* whether anonymous aggregates are clearly separate from event content;
* what the moderator is told before event content is deleted;
* whether the participant-facing recap page remains parked.

The output must be honest about retention: event content disappears 3 days after event end.

---

### Topic 8 — Admin, entitlement, abuse, and lockdown final check

Validate that the P1 decisions are complete enough for the tech interview and only revisit gaps.

Cover:

* invite-code entitlement semantics only if gaps remain;
* lockdown behavior: gates registration and event creation, never running or scheduled committed events;
* revocation behavior: next gated act only;
* emergency stop versus delete-all-data distinction;
* abuse doctrine: participants are moderator-managed; moderators/events are admin-managed;
* admin second auth layer if the product-level decision can be made now;
* whether any admin control needs to be added because of WIP-derived features.

Do not create team, organization, seat, or B2B account models.

---

### Topic 9 — Operations, resilience, and user-facing failure states

Resolve product-facing behavior for reliability without designing architecture.

Cover:

* reconnect and submission-save expectations as acceptance criteria;
* clear UI states for delayed save, retry, reconnecting, stale board, event full, and unknown/expired code;
* whether degraded modes are M5 only and depend on load-test evidence;
* which operational metrics are product/admin visible versus operator-only;
* how MicLine communicates best-effort/no-SLA without scaring moderators;
* what “committed events are sacred” means in failure copy and admin controls.

Keep advanced degraded-mode logic out of M1 unless absolutely necessary.

---

### Topic 10 — Milestone, backlog, and parking-lot freeze

Before writing final docs, review the complete milestone structure.

Use the current execution order:

1. M1 — Core product
2. M2 — Admin, entitlements, and stand-out features
3. M4 — UI overhaul / design system
4. M5 — Longevity / observability / operations
5. M3-if-ever — monetization, parked until later

Confirm:

* all accepted WIP candidates have a milestone;
* all rejected or parked ideas are clearly recorded;
* there is no M0 or M6 product milestone unless we explicitly create one;
* no old `presentation/session` model leaked back in;
* no participant access-control model was added accidentally;
* open questions are minimized and have clear consequences.

Then ask for my permission to generate the final documents.

---

## 6. Final output requirements

When the interview is complete, produce the final product artifacts in Markdown.

The output must include three complete documents, not only diffs:

1. `docs/product/product-brief.md`
2. `docs/product/feature-inventory.md`
3. `docs/product/decision-log.md`

These documents must incorporate:

* the confirmed P1 decisions that remain valid;
* all P2 changes and additions;
* superseded decisions where relevant;
* parked and rejected ideas where useful for future context;
* remaining open questions, only if unavoidable.

### 6.1 `product-brief.md` requirements

Include at least:

* one-liner;
* positioning;
* what MicLine is / is not;
* audience and market;
* personas;
* core model;
* event lifecycle;
* line modes;
* names and anonymity;
* joining;
* accounts;
* entitlements and invite codes;
* surfaces;
* product principles;
* limits and platform values;
* data lifecycle;
* design direction;
* language and tone;
* operations posture;
* final product boundaries.

### 6.2 `feature-inventory.md` requirements

Create a complete feature inventory grouped by milestone.

Each feature row must include:

* `id`
* `name`
* `one-sentence description`
* `milestone`
* `status`: `confirmed`, `assumed`, `open`, `parked`, or `rejected`
* optional short note if needed for ambiguity, dependency, or rejection reason.

Rules:

* Every accepted feature must have exactly one primary milestone.
* Do not duplicate the same feature under multiple names.
* Keep feature IDs stable where possible from the existing inventory.
* Add new IDs only when a P2 decision creates genuinely new scope.
* Mark rejected/parked WIP ideas clearly instead of deleting them silently if they are important context.
* Keep the inventory implementation-neutral enough for product use, but precise enough to support later Spec Kit specs.

### 6.3 `decision-log.md` requirements

Create a complete decision log grouped by topic.

Each entry must include:

* decision;
* why;
* rejected alternatives;
* status flag if applicable: `[ASSUMED]`, `[OPEN]`, `[PARKED]`, `[REJECTED]`, `[SUPERSEDED]`.

The decision log must clearly show when a P1 decision was confirmed, amended, or superseded during P2.

---

## 7. Final quality gates

Before printing the final documents, run this consistency check internally and fix issues:

* No accidental `Presentation` / `Session` product model remains.
* No participant account model exists.
* No participant access password, company-domain gate, or participant invite-code gate exists unless explicitly reopened.
* Zero-gate viewing remains intact.
* M1 remains small enough for a first real event.
* M2 semantics are precise enough for curated line, review mode, duplicates, upvotes, co-moderators, recap/export, and entitlements.
* Event-content retention remains 3 days after event end.
* Anonymous aggregates are separate from identifying event content.
* Open questions are few, explicit, and unavoidable.
* All accepted features appear in the feature inventory.
* All important rejected/parked ideas appear in the decision log or rejected/parked inventory section.
* Product docs do not smuggle in technical architecture choices reserved for the tech interview.

After producing the three documents, also print:

1. a concise `P2 change summary`;
2. the final feature-inventory table in chat;
3. the remaining `OPEN QUESTIONS` list, if any;
4. a short `Ready for tech interview` checklist.
```

### Revamp of milestone lineup

The artifacts `product-brief.md`, `feature-inventory.md` and `decision-log.md` were updated after the follow up product interview (P1.1), I reviewed them. I then used the prompt below with GPT 5.6 Sol and high effort to analyse the documents and propose an update to the milestone lineup. The lineup I got so far was dense (IMHO) and had to be updated (=more and smaller milestones).

```text
You are a senior SaaS product expert. Your job is to review the SaaS product idea I provide by carefully studying the attached documents. The provided documents product-brief.md, feature-inventory.md and decision-log.md are your most important source of information. They are the leading documents and all your work focuses on them. You do not speculate and you ask questions before responding if you need to clarify anything.

YOUR GOAL: Create a new milestone lineup by carefully analysing the current setup, splitting the work across the new milestones while paying special attention to under no circumstances loosing any content pieces of any attached document.

The goal with some more details:
1. Carefully review the provided leading documents
2. Propose a new lineup of milestones (there can be more milestones than already used in the documents - come up with a proposal)
3. Get my feedback to your proposal before moving on
4. After my milestone lineup approval, create an overview of the new milestones and assign work items accordingly
5. Get my feedback to your proposal (milestones + work items) before moving on
6. Update the attached documents carefully WHILE ENSURING TO NOT LOOSE ANY CONTENT
7. Perform a final deep check if you followed all ToDos listed here AND if you hit the stated goal to my full satisfaction, align with me on issues you found and after my approval correct your work if necessary
8. Request me to approve before writing the updated documents as markdown

After your work the updated documents will be used for the MicLine tech interview. You are preparing this interview with your work. Errors, inconsistencies and missing pieces of any kind must be spotted now and corrected before the tech interview can begin.
```

#### The outcome is a new milestone lineup

| Milestone                                             |                 Version | Release meaning                         |
| ----------------------------------------------------- | ----------------------: | --------------------------------------- |
| M1 — Deployable foundation                            |              **v0.1.0** | Deployable technical foundation         |
| M2 — Moderator identity and event setup               |              **v0.2.0** | Internal event-setup release            |
| M3 — Open-line event runtime                          |              **v0.3.0** | Internal end-to-end alpha               |
| M4 — Public-beta readiness                            |              **v0.4.0** | First responsible public beta           |
| M5 — Moderator control and data portability           |              **v0.5.0** | Moderator safety and export release     |
| M6 — Operator control plane                           |              **v0.6.0** | Normal administrative operations        |
| M7 — Emergency operations and recovery                |              **v0.7.0** | Exceptional-operations readiness        |
| M8 — Entitlements, invite codes and lockdown          |              **v0.8.0** | Operationally controlled public service |
| M9 — Curated moderation                               |              **v0.9.0** | Differentiated MicLine product          |
| M10 — Extended event toolkit                          |             **v0.10.0** | Broader moderator toolkit               |
| M11 — Design system, identity and experience overhaul |             **v0.11.0** | Product-wide visual release candidate   |
| M12 — Operational maturity and compliance             |              **v1.0.0** | First production-mature release         |
| M13 — Monetization, if ever                           | **v1.1.0**, provisional | Optional commercial extension           |

Because of this change, I had to update the MicLine build guide (docs/build-guide/), too. I used Claude code and pointed Fable 5 with max effort at the updated product artifacts.

The following prompt was used:

```text
Carefully review this repository of MicLine.app.
Especially important: docs/product/* & docs/build-guide/*
Initially the MicLine.app build guide was created to start from zero till releasing MicLine.app in production. By now some basic steps were already performed (see
PROGRESS.md) and the product artifacts (product-brief.md, feature-inventory.md, decision-log.md) were created and refined.

The product artifacts were just updated with a new milestone lineup.

YOUR GOAL: After carefully reviewing the product artifacts and the build guide as well as understanding the MicLine.app product, update the build guide carefully to
match the product artifacts (especially the new milestone lineup). The build guide and product artifacts must be aligned before the tech interview can begin.

Also propose updates (if applicable) to the build artifacts e.g. to remove outdated notices/hints e.g. to an old milestone setup.
Important: The product artifacts must not loose valuable product details.

ASK QUESTIONS IF REQUIRED AND WHEN REQUIRED instead of relying on assumtpions.
FINALLY carefully analyse your changes and by impersonating me, the creator of MicLine.app, read and understand the entire build guide to validate if the guide is
complete, free of errors and inconsistencies and can be used.
```

### Final product artifact review

I used the prompt below with Claude code and Fable 5 with max effort to run a critical product review before the tech interview begins.

```text
You are a senior SaaS product expert.

YOUR GOAL: Carefully review the current state of the MicLine.app product artifacts to understand the product deeply. You are required to critically review them as the final step before the tech interview begins.

Additional requirements for your work:
- You are required to be sceptical. Something does not make sense? Tell it out right!
- At any point you are required to ask questions if this supports you with your thinking/work.
- You need to ensure that after your review, the tech interview can begin. If you think details, hints, notes of any kind are missing in the product artifacts prompt me know to rectify this.
- If you find assumptions, analyse them and decide if you leave them alone or if you prompt me to convert an assumption to a decision.
- Apply your knowledge and best practices for creating SaaS product definitions to the MicLine.app documents and check if they are missing anything.
- Finally update the documents to state details about this final review step.
```

The product artifacts got updated based on my input. Also the build guide got adjusted.

Outcomes (2026-07-13, recorded as decision-log **§FR**): FR-1 public-beta risk acceptance · FR-2 concurrent-presence capacity model · FR-3 create-ahead default raised to 14 days (operator-configurable) · FR-4 truthfulness corrections · FR-5 tech-interview/spec-time registry additions · FR-6 new M4-gate scheduled decisions (success signals or no-targets stance, security-review pass, legal acknowledgment) · FR-7 reviewed-and-left-alone list. The product brief is now **Revision 4**. A follow-up prompt — *"Please run the same critical review for the build guide (docs/build-guide)."* — ran the identical pass over this guide the same day: its corrections are marked "(FR)" or dated 2026-07-13 in files 01/06/07/09 and PROGRESS.md, and the FR-6 items joined the [file 06 §8.4](06-m1-to-m4-plan.md) release gate.

**🛑 GATE 1:** read all three files. Fix wrong assumptions by telling Claude in the same session ("Change decision #12: …; update the files"). Commit: `docs: product definition v1 finalized`.

---

## §4 Phase 2 — Establish the tech context (the tech interview)

Spec Kit separates WHAT (specs) from HOW (plans). This phase creates the single HOW document every future `/speckit.plan` reads. Your inputs: `docs/inputs/tech-decisions.md` (ships with this guide — the distilled, pre-verified stack decisions), the three product artifacts (now on the M1–M13 lineup), and — crucially — the decision log's **"Tech interview (binding inputs)"** registry, which the P2/P3 product phases loaded with concrete obligations for exactly this phase. Two of them are non-negotiable framing (decisions T10-2/P3-2): the interview **starts by fully outlining the complete Cloudflare + GitHub Actions setup**, and the foundation gets **revisited once the M3 / `v0.3.0` end-to-end alpha exists**.

```text
PROMPT P2 — tech interview → tech context
[MODEL: Fable 5] [EFFORT: high] [SESSION: fresh]

Read docs/inputs/tech-decisions.md, docs/product/product-brief.md,
docs/product/feature-inventory.md, docs/product/decision-log.md (at minimum
the P3 and FR sections and the "Open questions & scheduled decisions"
registry at the end), docs/build-guide/06-m1-to-m4-plan.md §8.0, and — if present —
docs/inputs/legacy-spec.md. The product docs OVERRIDE the technical inputs
wherever they conflict; list every conflict you find in a "Superseded"
section at the end. Note that tech-decisions.md predates the final product
interviews: its question-line sketch still contains pre-P2 semantics
(votes/pre-moderation as core, no skip-stub, no one-active-item rule) —
the feature inventory and decision log win on every such conflict.

Then, for each Cloudflare product in the stack (workers, d1, durable-objects,
kv, email-service, turnstile), fetch developers.cloudflare.com/⟨product⟩/
llms.txt and skim the most relevant pages; reconcile what you read against
tech-decisions.md and note every correction.

Conduct the interview before writing anything (P1 rules: one topic at a
time, batches ≤5, be critical, ASSUMED where I delegate):
  0. FIRST — binding requirement T10-2/P3-2: fully and precisely outline the
     complete Cloudflare + GitHub (Actions) setup end to end (accounts,
     zones, workers, environments, resources, secrets, pipelines), so I see
     the whole machine before any detail decision.
  1. Then work through EVERY item of the decision log's "Tech interview
     (binding inputs)" list: admin-wall mechanism criteria (OQ1 — Cloudflare
     Access vs TOTP vs passkey; pricing/limits/exit cost; the mandatory
     Cloudflare-account-only reset), board capability-token design (no event
     ID/join code exposure, no enumeration), central platform-config +
     feature-flag mechanism (OQ8) and the global log-level control (FND-06),
     the email-template registry + react.email evaluation, magic-link
     architecture depth (OQ7), rate-limit keys incl. rescheduling, emergency
     export-download isolation, and the persistence + scheduled-purge
     architecture. Decide what is decidable now; anything genuinely owned by
     a later milestone spec gets recorded as such, not silently dropped.

Then produce exactly three artifacts:

1. docs/product/constitution-input.md — enduring PRINCIPLES and CONSTRAINTS
   only (public-repo/zero-secrets; Cloudflare-first; correctness invariants
   of the two-mode-shaped line state machine; one entitlement seam; testing
   gates; privacy/data-lifecycle incl. the committed-events-are-sacred and
   zero-gate rules). No tech choices, no schemas.

2. docs/engineering/tech-context.md — the curated HOW knowledge every future
   /speckit.plan must read: pinned stack table; architecture (single Worker,
   Hono for /api+/ws, React Router SSR fallthrough, shared service layer so
   loaders and API routes call the same functions); data-model draft marked
   "draft — authoritative version emerges per-feature"; realtime design
   (DO-per-event, WebSocket Hibernation) honoring the binding reconnect
   acceptance criteria (decision T9-2); the open-line state machine, built
   two-mode-shaped for M9; auth design (magic links, HMAC cookies); abuse
   stack; platform-config/flag + log-level mechanism; email-template
   registry; monorepo layout; secrets matrix; environment model (DEV/PRD —
   see docs/build-guide/03-github-operations.md); CI/CD; test strategy.
   Fold the "Architecture rules", abuse/hardening defaults, and "Verified
   findings" sections of tech-decisions.md into this file — corrected per
   your doc check and the interview — so that NO later plan ever needs to
   open tech-decisions.md again. End with a "Verify at implementation time"
   list for every external fact.

3. docs/product/feature-briefs/ — one file per M1–M4 work package. Default
   cut = docs/build-guide/06-m1-to-m4-plan.md §8.0 (F000–F007 legacy +
   F008–F013 provisional); challenge that cut where the architecture
   suggests a better one, but every M1–M4 feature-inventory ID must land in
   exactly one brief, and each brief names its milestone, target version,
   and the inventory IDs it implements. Reconcile against the inventory's
   M1–M4 rows; flag misfits. Each brief ≤ 1 page: user value, in-scope,
   out-of-scope, dependencies — WHAT/WHY only; any HOW you're tempted to
   write goes into tech-context.md.

Print the Superseded list, the interview's decisions (incl. ASSUMED ones),
the final work-package cut, and the file tree you created.
```

### Tech interview artifact review

I read the created `docs/engineering/tech-context.md`, took notes and used the prompt below with Claude code and Fable 5 with max effort to run a critical review before moving on with `§5 Phase 3 — Constitution`.

```text
You are a senior SaaS and Cloudflare expert. Scan this repo to fully understand the product MicLine.app
Important sections of the docs are:
- docs/build-guide (we are currently at 04-product-definition.md -> §4 Phase 2 — Establish the tech context (the tech interview))
- docs/product/*
- docs/engineering/tech-context.md

GOAL: After understanding the product MicLine.app you are a CRITICAL REVIEWER of the tech interview artifacts. You are thoroughly reviewing the created artifacts. There is no need to critique anything for the sake of doing it. You must always fund and explain your requested changes and work together with the operator until all your findings are evaluated (=accepted or rejected). The result of your work are validated tech interview artifacts ready to be used for the spec-driven development approach (build-guide ->
05-feature-loop.md). The entire collection of the MicLine build-guide, the product artifacts and the tech interview artifacts need to be sound and aligned to avoid confusion and issues during the next steps.

After reviewing the current state of the docs, the operator took some notes - incorporate them for your work:
- Does the current setup protects itself from malicious users (be it a moderator, a future co-moderator or a participant)? Just one exampple: Never trust data from the user of the product (like submitted strings) etc.
- An admin should be able to link a voucher he creates to an email address. This way he can enforce that once he hands a voucher to a person, that this person must use their own (set by the admin) email address while signing up as a moderator (this way voucher codes can not be transferred between people).
- We need to check what admin statistics are being stored and what might be missing.
- The docs (as an example but not necessarily limited to the build-guide) need to have a section with intructions for the operator (and agents) to follow what precisely needs to be done to introduce a new feature to MicLine (while development is ongoing of if the development is already done/all currenty planned features are implemented). It needs to be outlined from the very beginning (the feature described by the poerator the very first time) till it is fully implemented.
- SEO must be properlysetup for the landing page and its sub-pages (refer to SEO best practices)
- 2 auth modes for the admin UI: (1) Password and (2) Cloudflare Access. To (1) password is used for quick start, wrangler secret admin can put, signed 5-day session cookie, timing safe checks, throttled failures, admin UI will nudge towards Cloudflare Access. (2) Cloudflare Access recommended option, verified in-worker on every request, setting it up disables the password login automatically - CF Access always wins. If no option is configured: /admin fails closed with setup instructions. A one-click deploy should always fail with an obvious error instead of a running app with no auth for /admin.
- MicLine needs to have proper protection against search engines to avoid certain pages being indexed
- Deployment config var to enable/disable a /health endpoint which only returns HTTP 200 if MicLine core services are poerational (like the worker + D1 + maybe KV?). Must not be behind auth.
- 2 MicLine deployment options (aside the mainly used GitHub pipeline deployment): (1) via Cloudflare deploy button and (2) via the CLI (maybe npm?). For CLI: npm scaffolds a fresh copy - repo downloaded, dependencies installed, fresh git history - and prints the setup steps.

At any point in time you are required to ask questions if it helps you with your work instead of only relying on assumptions.

After evaluating the operators notes, making a plan to integrate them and executing this plan, perform a thorough review of your work before moving on. Introduce fixes if required.

At the end of this review you scan your work once more to check if any changes you made caused conflicts (from content to markdown links), state them clearly and fix them. Finally you create a summary of your work.
```

Outcomes (2026-07-14, every finding operator-evaluated; full amendment record in **tech-context §23**): security assessment passed with three accepted findings — S1 `ENVIRONMENT` becomes `local`/`dev`/`production` so the admin-wall bypass and F001 auth stub can never run on deployed envs · S2 the magic-link per-email send cap pulled forward into F002/M2 (closes the M2→M4 window with a live but unthrottled PRD auth endpoint) · S3 per-environment WS-Origin allowlist. Operator notes incorporated: DEV/PRD email sending domains split (`dev.micline.app` / `micline.app`, separate onboardings — 02 §2.7 row 2 rewritten) · binding resource-naming (`micline` prefix) + Resource-Tagging (`project:micline`, `env:*`) rule for the shared Cloudflare account · config-key descriptions + the admin-UI ~90 s propagation notice · audit-log retention and emergency-export link validity confirmed runtime-configurable · dependency-budget rule · test-per-feature rule (constitution-input IX + CONTRIBUTING) · Local Explorer + workers-sdk/miniflare notes. New committed docs: `docs/engineering/runcost-estimate.md` and `docs/runbooks/` `security-incident-response.md` · `maintenance.md` · `resource-bootstrap.md` (stub — F000 completes it).

### Introduction of additional features + critical review

I instructed Claude code and Fable 5 with max effort to evaluate some more features/tweaks I had in mind (list in prompt) and to, afterwards, run another critical review before moving on with `§5 Phase 3 — Constitution`.

```text
You are a senior SaaS and Cloudflare expert. Scan this repo to fully understand the product MicLine.app
Important sections of the docs are:
- docs/build-guide (we are currently at 04-product-definition.md -> §4 Phase 2 — Establish the tech context (the tech interview))
- docs/product/*
- docs/engineering/tech-context.md

GOAL: After understanding the product MicLine.app you are a CRITICAL REVIEWER of the tech interview artifacts. You are thoroughly
reviewing the created artifacts. There is no need to critique anything for the sake of doing it. You must always fund and explain your
requested changes and work together with the operator until all your findings are evaluated (=accepted or rejected). The result of your
work are validated tech interview artifacts ready to be used for the spec-driven development approach (build-guide ->
05-feature-loop.md). The entire collection of the MicLine build-guide, the product artifacts and the tech interview artifacts need to be
sound and aligned to avoid confusion and issues during the next steps.

The operator created the following list of features/tweaks/ideas he had in mind to be challenged now before the constitution is being created as the next step.
Here are the notes he took - incorporate them for your work (assessment can, if required, happen now via a structured interview, like for the tech interview):
- Does the current setup protects itself from malicious users (be it a moderator, a future co-moderator or a participant)? Just one exampple: Never trust data from the user of the product (like submitted strings) etc.
- An admin should be able to link a voucher he creates to an email address. This way he can enforce that once he hands a voucher to a person, that this person must use their own (set by the admin) email address while signing up as a moderator (this way voucher codes can not be transferred between people).
- We need to check what admin statistics are being stored and what might be missing. Lets run through the entire list.
- The docs (as an example but not necessarily limited to the build-guide) need to have a section with intructions for the operator (and agents) to follow what precisely needs to be done to introduce a new feature to MicLine (while development is ongoing and if the development is already done/all currently planned features are implemented). It needs to be outlined from the very beginning (the feature described by the poerator the very first time) till it is fully implemented. In short: There must be a funnel the operator can follow to extend/update MicLine in the future.
- SEO must be properly setup for the landing page and its sub-pages (refer to SEO best practices)
- 2 auth modes for the admin UI (ensure this does not collide with the current outline): (1) Password and (2) Cloudflare Access. To (1) password is used for quick start, wrangler secret admin can put, signed 5-day session cookie (this default needs to be stated for the operator to know, ideally right next to the step instructing him on how to setup (1)), timing safe checks, throttled failures, admin UI will nudge towards Cloudflare Access. (2) Cloudflare Access recommended option, verified in-worker on every request, setting it up disables the password login automatically - CF Access always wins. If no option is configured: `/admin` fails closed with setup instructions. A one-click deploy should always fail with an obvious error instead of a running app with no auth for `/admin`.
- MicLine needs to have proper protection against search engines to avoid certain pages being indexed
- Deployment config var to enable/disable a `/health` endpoint which only returns HTTP 200 if MicLine core services are operational (like the worker + D1 + maybe KV?). Must not be behind auth.
- 2 MicLine deployment options (aside the mainly used GitHub pipeline deployment): (1) via Cloudflare deploy button and (2) via the CLI (maybe npm?). For CLI: npm scaffolds a fresh copy - repo downloaded, dependencies installed, fresh git history - and prints the setup steps.
- For further activities within this repository, we should have a central place for humans and especially for agents to lookup where certain details/content can be found in this repo (because just the docs are already extensive and as of now there is not a single line of code written yet). Use industry known best practices to find a proper approach and get it setup for MicLine.app

At any point in time you are required to ask questions if it helps you with your work instead of only relying on assumptions.

At the end of this review you carefully scan your work once more to check if any changes you made caused conflicts (from content to markdown links), state them clearly and fix them. Finally you create a summary of your work if you consider everything to be ready for §5 Phase 3 — Constitution.
```

Outcomes (2026-07-15, every note operator-evaluated — product record: decision-log **§AF**; technical record: tech-context **§24**): all ten notes accepted, two of them as amendments to previously closed decisions. **AF-7** the admin wall becomes dual-mode (`ADMIN_PASSWORD` quick-start with a signed 5-day session, or Cloudflare Access — recommended and always winning; fails closed with setup instructions when unconfigured) **and the wall is now the admin auth**: the operator's review surfaced that magic-link login has no place on `/admin` — the admin is not a product user, `users.role` was dropped, and the control plane no longer depends on MicLine's own email pipeline (amends OQ1's 2026-07-14 closure and D1/T8-1). **AF-9** self-hosting ratified as a repo-side path at M12 (new **OPS-07**: Deploy-to-Cloudflare button + C3 `--template` scaffold — deliberately no own npm package — + SELF_HOSTING guide; forward-compat demands binding on M1–M4). Further: **AF-8** `/health` endpoint (new **FND-07**, M1/F008, deploy-time var); **AF-2** voucher email lock (locked ⇒ single-use, M8); **AF-3** three-store admin-stats model with the keyless `ops_counters` catalog written from each signal's first milestone; **AF-5/AF-6** SEO checklist + default-deny `X-Robots-Tag` indexing (F000 baseline, F007 allowlists marketing on PRD); **AF-1** hardening addenda (export injection rules for M5, body-size caps, CSRF stance, bidi strip, Referrer-Policy); **AF-4** the intake funnel ([file 05 §7.0](05-feature-loop.md)); **AF-10** the documentation map at `docs/README.md`. The product brief is now **Revision 5**.

### Pre-Constitution Hostile Review

After incorporating another set of features and thoughts and reviewing the entire document landscape, I used GPT 5.6 Sol with high effort to create the following prompt to instruct Claude Fable 5 with max effort to act as a hostile senior reviewer as the final quality gate.

```text
You are working inside the local repository for **MicLine.app**.

You are the **Senior Hostile Reviewer** responsible for the final adversarial review before the operator proceeds to:

`docs/build-guide/04-product-definition.md` → `§5 Phase 3 — Constitution`

Your job is not to validate the operator’s confidence. Your job is to determine whether that confidence is deserved.

You must inspect the repository, interrogate the product and technical definition, challenge the operator, correct the documentation, and keep going for as many rounds as necessary until the document collection is genuinely safe to use as the source for the product constitution.

No sugar coating. No congratulatory filler. No ceremonial approval. No critique for theatrical effect. Every finding must be evidence-based and consequential.

“Hostile” describes your posture toward assumptions, documents, decisions, and failure modes. It does not permit insults, aggression toward the operator, or performative negativity.

<mission>

Produce both of these outcomes:

1. A **sound, internally consistent, traceable, constitution-ready repository document set**.
2. A **confident operator who actually understands and can defend the product model, constraints, trade-offs, risks, and unresolved decisions**.

The review is complete only when the repository can honestly receive this verdict:

`READY FOR PHASE 3 — CONSTITUTION`

Until every readiness criterion in this prompt is satisfied, the verdict is:

`NOT READY FOR PHASE 3`

You have no target number of questions, findings, review rounds, or document changes.

Continue as long as necessary.

Do not stop because:

- previous reviews were called “final”, “critical”, or “approved”;
- the operator previously accepted a decision;
- many questions have already been asked;
- the session has become long;
- the remaining defect is inconvenient;
- most documents agree;
- a claim sounds reasonable;
- a model wrote the existing text confidently;
- or the repository’s progress tracker currently points toward the constitution.

</mission>

<phase_boundary>

This is a **pre-constitution review**.

You must not:

- run `/speckit.constitution`;
- create or rewrite the final `.specify/memory/constitution.md`;
- begin feature specification, planning, tasks, or implementation;
- write application code;
- silently add product scope;
- commit, push, merge, rebase, tag, publish a release, or deploy;
- or declare the constitution phase started.

You may improve `docs/product/constitution-input.md` and map constitution-ready principles, conflicts, evidence, exceptions, and enforcement consequences.

You must stop at the constitution threshold.

</phase_boundary>

<operating_posture>

Treat every material statement as a claim that must be classified and tested.

This includes statements from:

- the operator;
- prior Claude sessions;
- review summaries;
- authoritative documents;
- raw inputs;
- external documentation;
- comments;
- examples;
- and this prompt’s repository-specific hypotheses.

The operator is the product decision-maker, but not an infallible source of facts.

Do not equate:

- confidence with evidence;
- approval with correctness;
- repetition with independent confirmation;
- authority with truth;
- implementation detail with product law;
- “MVP” with permission to leave core behavior undefined;
- “best effort” with an observable guarantee;
- “later” with a legitimate deferral;
- or “GDPR” wording with legal validity.

When the operator answers a question:

1. Extract the exact decision being made.
2. Separate the decision from supporting explanation and assumptions.
3. Compare it against every relevant authoritative document.
4. test it against at least one hostile or failure scenario.
5. identify downstream product, technical, legal, security, privacy, operational, milestone, and constitutional consequences;
6. identify every document that must change;
7. challenge vague language, hidden exceptions, and unverified external facts;
8. ask a narrower follow-up when the answer is incomplete;
9. close the finding only after the accepted decision is documented and verified.

Do not reopen a settled decision without a concrete reason.

A settled decision may be reopened when there is:

- contradictory current text;
- an undefined boundary or exception;
- a material unconsidered failure mode;
- a security, privacy, accessibility, legal, operational, or economic conflict;
- changed external evidence;
- an impossible combination of requirements;
- a milestone or dependency contradiction;
- a constitution-level collision;
- or a downstream artifact that cannot implement the decision faithfully.

</operating_posture>

<progress_integrity>

Claude Fable 5 is capable of long-running work. Use that capability without making false progress claims.

During long work:

- provide brief, evidence-based progress updates at meaningful phase boundaries;
- report only work actually completed;
- name artifacts inspected, commands run, findings confirmed, and patches applied;
- never invent percentages;
- never say a review is complete while uninspected corpus remains;
- never imply a subagent result has been verified until you independently checked it;
- never report a command as passed unless you ran it and inspected the result.

Use the repository and review ledger as memory. Do not rely on conversational memory alone.

</progress_integrity>

<repository_safety>

Before substantive review, establish a baseline:

```bash
pwd
git rev-parse --show-toplevel
git branch --show-current
git rev-parse HEAD
git status --short
git log -1 --oneline
git remote -v
```

Then:

1. Confirm that the working directory is the MicLine.app repository root.
2. Record the branch and baseline commit.
3. Identify all pre-existing tracked and untracked changes.
4. Preserve operator work exactly.
5. Do not overwrite a changed file until you have inspected its diff and understood whether the change belongs to this review.
6. Re-run `git status --short` before every editing wave and at the end.
7. Never use `git reset --hard`, `git clean`, force checkout, force push, history rewriting, destructive restore commands, or an equivalent shortcut.
8. Never stage or commit unless the operator gives a later, explicit instruction outside this prompt.
9. Never use broad staging commands such as `git add .` during this review.
10. Do not mutate GitHub, Cloudflare, package registries, DNS, email configuration, or any other remote system.
11. Read-only remote queries are allowed only when needed for verification.
12. Do not expose secrets in commands, output, shell history, logs, fixtures, documentation, or review notes.

Maintain a temporary ledger at:

`/tmp/micline-pre-constitution-hostile-review.md`

The ledger is review state, not a repository artifact. Do not commit it.

The ledger must survive context pressure and contain enough information to resume accurately:

- baseline;
- corpus inventory;
- findings;
- questions asked;
- operator decisions;
- document amendments;
- verification results;
- unresolved work;
- and the next action.

</repository_safety>

<first_action>

Do not begin with broad onboarding questions.

Inspect first.

Start with:

1. `docs/README.md`
2. `docs/build-guide/PROGRESS.md`
3. `docs/build-guide/04-product-definition.md`, especially the current Phase 2 handoff and `§5 Phase 3 — Constitution`
4. root `CLAUDE.md`
5. root `CONTRIBUTING.md`
6. root `.gitignore`
7. the authoritative product and technical artifacts

Only ask the operator questions after you have enough evidence to make each question precise.

</first_action>

<mandatory_corpus>

Use `git ls-files` as the base inventory. Do not rely on a manually remembered file list.

Review every tracked artifact that can affect product definition, technical definition, repository governance, agent behavior, delivery, security, operations, or the constitution.

At minimum, review:

- root `README.md`;
- root `CLAUDE.md`;
- root `CONTRIBUTING.md`;
- root `LICENSE`;
- root `.gitignore`;
- every file under `docs/build-guide/`;
- every file under `docs/product/`;
- every feature brief under `docs/product/feature-briefs/`;
- every existing epic under `docs/product/epics/`;
- every file under `docs/engineering/`;
- every file under `docs/runbooks/`;
- every file under `docs/inputs/`, while treating it as provenance rather than authority;
- relevant files under `.specify/`;
- relevant files under `.claude/`;
- relevant files under `.github/`;
- tracked scripts, templates, configuration, and metadata;
- any current local changes;
- and any other tracked file that asserts requirements, commands, conventions, lifecycle behavior, security controls, or release behavior.

For each relevant file record:

- path;
- purpose;
- authority category;
- current, historical, generated, planned, placeholder, or provenance-only status;
- normative assertions;
- dependencies;
- dependants;
- cross-checks performed;
- and any unresolved concern.

A file is not “reviewed” merely because it was opened.

A file counts as reviewed only after its material claims have been compared with the documents that own or depend upon the same subject.

</mandatory_corpus>

<authority_model>

Begin with the current local `docs/README.md` and obey its authority map.

Do not invent a global “newest file wins” rule.

The current repository distinguishes at least these authorities:

- `docs/product/feature-inventory.md` owns what ships, milestone assignment, status, and target version.
- `docs/product/decision-log.md` owns why a decision was confirmed, assumed, opened, scheduled, parked, rejected, amended, or superseded.
- `docs/product/product-brief.md` owns the product model, positioning, actors, principles, behavior, and lifecycles.
- The product brief’s data-lifecycle model requires constitutional scrutiny.
- `docs/engineering/tech-context.md` owns implementation architecture and technical mechanisms.
- `docs/product/constitution-input.md` owns candidate enduring constraints feeding the constitution.
- `docs/product/feature-briefs/` owns per-work-package WHAT and WHY for the applicable engineering cut.
- `docs/build-guide/` owns the operating process, gates, prompts, and delivery workflow.
- `docs/build-guide/PROGRESS.md` alone owns the current workflow position.
- `docs/runbooks/` owns executable operational procedures.
- `docs/inputs/` is provenance only unless the current documentation map explicitly says otherwise.
- The constitution owns constitutional law only after Phase 3 has legitimately completed.

When documents overlap:

1. identify the exact question;
2. identify the authority that owns that question;
3. distinguish contradiction from complementary detail;
4. preserve historical rationale;
5. correct the owning artifact;
6. update every dependent artifact;
7. mark superseded material explicitly;
8. never leave two current-looking answers.

Authority determines where the corrected answer belongs. It does not make the current answer automatically correct.

</authority_model>

<classification>

Classify each material statement or decision as exactly one of:

- `DECIDED`
- `ASSUMED`
- `OPEN`
- `SCHEDULED`
- `PARKED`
- `REJECTED`
- `SUPERSEDED`
- `EXTERNAL-FACT`
- `LEGAL-VERIFICATION-REQUIRED`

Do not blur categories.

In particular:

- a repeated assumption remains an assumption;
- an operator-approved assumption becomes decided only when the actual decision is explicit;
- a scheduled decision is not decided;
- a parked item is not active scope;
- a rejected item must not reappear indirectly;
- an external platform fact is not a product decision;
- a legal conclusion cannot be manufactured by an LLM;
- a current technical mechanism does not automatically belong in the constitution;
- an enduring product rule must not be dismissed as “implementation detail” merely because it is difficult.

Every deferred decision must specify:

- what is deferred;
- why deferral is safe;
- owner;
- exact trigger or milestone;
- evidence required;
- destination artifact;
- fallback if unresolved;
- and why it does not alter constitutional meaning.

A decision is not safe to defer when it changes:

- public promises;
- user rights;
- actor authority;
- core states or transitions;
- trust boundaries;
- data categories or retention;
- security posture;
- milestone feasibility;
- or the meaning of a constitutional principle.

</classification>

<finding_ledger>

Assign stable IDs:

- `PC-B###` — blocker
- `PC-H###` — high
- `PC-M###` — medium
- `PC-L###` — low

Each finding must contain:

- ID
- title
- severity
- status
- review domain
- authoritative owner
- exact evidence with paths and headings or line ranges
- conflicting, incomplete, unsafe, or unsupported assertion
- why the current text is insufficient
- concrete hostile or failure scenario
- affected actors
- affected milestones and versions
- affected artifacts
- decision required
- reviewer recommendation
- credible alternatives
- trade-offs
- operator answer
- reviewer challenge
- final accepted decision
- required amendments
- verification evidence
- residual risk
- closure status

Allowed statuses:

- `DISCOVERED`
- `QUESTIONED`
- `ANSWER-INCOMPLETE`
- `DECISION-ACCEPTED`
- `RISK-ACCEPTED`
- `PATCHED`
- `VERIFICATION-FAILED`
- `VERIFIED`
- `LEGITIMATELY-DEFERRED`
- `REJECTED-AS-NOT-A-DEFECT`

Severity definitions:

### BLOCKER

A defect that can invalidate the product model or constitution, create incompatible requirements, materially misstate user rights or data behavior, leave a core lifecycle undefined, or make the release path unsafe or impossible.

### HIGH

A likely security, privacy, abuse, availability, accessibility, operational, traceability, or milestone defect with material user or business impact.

### MEDIUM

Ambiguity or incompleteness likely to cause specification churn, inconsistent implementation, support burden, or avoidable rework.

### LOW

A non-material terminology, link, provenance, formatting, or editorial defect.

Do not downgrade severity because a fix is inconvenient.

Do not inflate severity to sound hostile.

</finding_ledger>

<independent_review_passes>

Perform independent review passes with deliberately different lenses.

Use read-only subagents where available so large-corpus exploration does not pollute the coordinating context. Each subagent must receive an explicit corpus, attack surface, and output schema.

Use at least these passes:

1. **Authority, provenance, and cross-document traceability**
2. **Product identity, actors, journeys, state machines, and lifecycle completeness**
3. **Security, abuse, privacy, retention, and legal-claim hygiene**
4. **Real-time and distributed-state correctness**
5. **Operations, recovery, reliability, cost, and single-operator feasibility**
6. **Milestones, SemVer, work packages, release gates, and dependency feasibility**
7. **Accessibility, internationalization, hostile content, and user-facing truthfulness**
8. **Repository governance, Git safety, CLI operations, secret handling, and `.gitignore`**
9. **Constitution suitability: enduring principle versus implementation mechanism**

Subagents are reviewers, not authorities.

The coordinating reviewer must:

- inspect high-severity evidence directly;
- reconcile disagreements;
- remove duplicates without losing evidence;
- challenge weak subagent findings;
- reject unsupported findings;
- and remain responsible for the final verdict.

If subagents are unavailable, perform the passes sequentially and explicitly reset the evaluation lens between passes.

</independent_review_passes>

<mandatory_attack_surfaces>

Review all of the following. This is a floor, not a ceiling.

## 1. Product identity and boundaries

Determine whether the repository gives one coherent, testable answer to:

- What exact problem does MicLine solve?
- For whom?
- In which event formats?
- What is the design center?
- What is supported but not targeted?
- What is explicitly not the product?
- What public promise is created by “the live question line for every event”?
- What does “every event” exclude?
- What differentiates MicLine from generic Q&A, chat, polling, webinar, event-management, and moderation products?
- Which principles are enduring?
- Which statements are launch tactics or marketing aspirations?
- Which product claims can be tested?

Attack vague terms including:

- fair;
- transparent;
- simple;
- live;
- real-time;
- reliable;
- anonymous;
- private;
- GDPR-clean;
- zero-gate;
- no tracking;
- no cookie banner;
- committed;
- sacred;
- production-ready;
- self-hostable;
- all features;
- best effort;
- and every event.

Require operational definitions whenever ambiguity changes behavior.

## 2. Actors, identity, authority, and trust

Identify every human and machine actor.

For each actor establish:

- identity mechanism;
- authentication strength;
- authorization;
- capability possession;
- session lifetime;
- revocation;
- destructive powers;
- data visibility;
- impersonation risk;
- recovery path;
- and abuse potential.

Include at least:

- anonymous viewer;
- participant;
- named participant;
- moderator;
- event owner;
- future co-moderator;
- account holder;
- operator/admin;
- banned user;
- disconnected or stale client;
- board viewer;
- malicious browser;
- automated bot;
- compromised moderator;
- compromised administrator;
- email recipient;
- invite-code recipient;
- self-hosting deployer;
- platform service;
- and CI/CD identity.

Never assume an actor is benevolent because the documents call them “moderator”, “host”, “admin”, or “operator”.

## 3. End-to-end journeys

Trace every important journey from entry to terminal state, including:

- account creation and login;
- event creation;
- scheduling and rescheduling;
- opening;
- joining;
- lobby;
- full-event denial;
- reconnect;
- submission;
- retry;
- withdrawal;
- leave and rejoin;
- moderation;
- skip;
- answer;
- undo;
- announcement;
- hide;
- close intake;
- review;
- decline;
- merge;
- reorder;
- co-moderation;
- export;
- recap;
- account deletion;
- inactivity purge;
- ban;
- force-end;
- emergency stop;
- emergency resume;
- event purge;
- account purge;
- entitlement redemption and revocation;
- credential rotation;
- incident response;
- deployment during a live event;
- and self-hosting setup.

For every journey ask:

- What can happen before it?
- What may happen concurrently?
- What can fail halfway?
- What is idempotent?
- What is retried?
- What is irreversible?
- What does each actor see?
- What is persisted?
- What is deleted?
- What survives?
- Who can repair it?
- What happens when that actor is absent?

## 4. State machines and time

Reconstruct the actual state machines rather than trusting labels.

Cover:

- event;
- question;
- participant session;
- moderator session;
- board capability;
- account;
- admin wall;
- voucher;
- entitlement;
- email delivery;
- export;
- purge;
- emergency controls;
- and feature flags.

For every transition require:

- preconditions;
- actor authority;
- command;
- idempotency behavior;
- persisted result;
- client-visible result;
- concurrency behavior;
- denial behavior;
- retry behavior;
- diagnostic evidence;
- and terminal-state rules.

Test temporal semantics:

- source timezone;
- storage timezone;
- display timezone;
- daylight-saving changes;
- ambiguous and nonexistent local times;
- clock skew;
- create-ahead windows;
- reschedule anchors;
- automatic start;
- automatic end;
- grace periods;
- retention anchors;
- emergency pause and resume;
- and events spanning midnight or timezone changes.

## 5. Feature, milestone, SemVer, and work-package integrity

Prove mechanically where practical that:

- active feature IDs are unique;
- each active feature has one authoritative milestone;
- each active feature has one authoritative target version;
- every M1–M4 feature maps to exactly one F000–F013 work package;
- every work-package reference resolves;
- every feature-brief inventory ID exists;
- no active item is orphaned;
- no item is duplicated under multiple names;
- parked and rejected work has not leaked into current scope;
- cross-milestone extensions do not contradict base features;
- work packages do not conceal unrelated scope;
- release gates do not require functionality scheduled later;
- and milestone outcomes are honest about what a user can actually do.

Keep these namespaces distinct:

- product feature IDs;
- F000–F013 engineering work packages;
- milestone IDs;
- decision IDs;
- review-finding IDs;
- Spec Kit feature numbers;
- email IDs;
- and external requirement identifiers.

Flag any naming collision or likely agent/operator confusion.

## 6. Data map, privacy, retention, and legal hygiene

Build a complete data inventory.

For every datum identify:

- data subject;
- collection point;
- purpose;
- necessity;
- storage;
- replication;
- cache;
- logs;
- counters;
- analytics or aggregate exposure;
- email-provider exposure;
- export exposure;
- backup or point-in-time-recovery exposure;
- retention anchor;
- deletion mechanism;
- deletion exception;
- operator visibility;
- moderator visibility;
- participant visibility;
- and self-hosting implications.

Include at least:

- names;
- pseudonyms;
- questions and free text;
- participant session identifiers;
- cookies;
- IP-derived signals;
- geolocation and country;
- email addresses;
- magic-link material;
- moderator sessions;
- admin sessions;
- vouchers and email locks;
- entitlements;
- event metadata;
- event content;
- moderator actions;
- admin audit actions;
- operational counters;
- security logs;
- exports;
- emails;
- feedback content;
- backups;
- and emergency snapshots.

Challenge absolute privacy statements.

Examples requiring proof:

- no personal data;
- anonymous;
- no tracking;
- no cookie banner required;
- GDPR-clean by construction;
- everything identifying dies;
- deleted after three days;
- no durable action trails;
- and nothing beyond a session cookie.

Do not act as legal counsel.

For each legal claim:

- separate product behavior from legal conclusion;
- identify professional verification required;
- prefer testable behavioral commitments;
- prevent unsupported legal certainty from entering the constitution;
- and flag laws or platform terms that need current verification.

## 7. Security and abuse

Assume every external input and capability is hostile.

Review:

- capability URLs;
- join codes;
- board tokens;
- magic links;
- invite codes;
- admin credentials;
- cookies;
- HTTP APIs;
- WebSockets;
- CSRF;
- origin checks;
- replay;
- enumeration;
- brute force;
- rate-limit evasion;
- shared NAT;
- bot-defense failure;
- multiple tabs;
- stolen sessions;
- rotation and revocation;
- content injection;
- SQL injection;
- XSS;
- CSV formula injection;
- HTML export injection;
- email injection;
- control characters;
- Unicode bidi controls;
- oversized bodies;
- filenames;
- open redirects;
- referrer leakage;
- search indexing;
- logs containing secrets;
- dependency and CI compromise;
- hostile pull requests;
- and denial of service.

For every control establish:

- exact threat;
- what the control does;
- what it does not do;
- preventive, detective, or recovery role;
- false-positive behavior;
- shared-network behavior;
- dependency-failure behavior;
- fail-open or fail-closed posture;
- override authority;
- and whether public product claims remain truthful during degradation.

## 8. Real-time and distributed correctness

Challenge:

- source of truth;
- ownership of mutable event state;
- serialization;
- command routing;
- ordering;
- sequence numbers;
- duplicate delivery;
- lost responses;
- reconnect;
- stale clients;
- split brain;
- eventual consistency;
- scheduled work;
- alarms;
- queues;
- email side effects;
- purge side effects;
- and deployment behavior.

At minimum test these scenarios:

- A command succeeds but its response is lost.
- A participant retries after reconnect.
- A moderator uses two tabs.
- Two moderators mutate the same line.
- A participant withdraws while a moderator calls the item.
- Undo races with the next transition.
- A question is merged while another action targets it.
- The event ends with commands in flight.
- Emergency stop activates mid-command.
- A deploy occurs during a full event.
- The authoritative process restarts.
- Storage succeeds but broadcast fails.
- Some clients receive an update and others do not.
- A scheduled transition fires twice.
- A purge partially succeeds.
- Email sends but delivery state is not recorded.
- A reconnect occurs after a presence slot was released.

Do not accept “Cloudflare handles it” without the exact invariant MicLine depends on.

## 9. Capacity and fairness

Define:

- capacity;
- concurrent presence;
- active;
- connected;
- disconnected;
- grace period;
- stale;
- admitted;
- full;
- waiting;
- read-only;
- multiple tabs;
- and fairness.

Attack the model under:

- mobile sleep;
- flaky Wi-Fi;
- reconnect storms;
- multiple tabs;
- shared cookies;
- cleared cookies;
- browser refresh;
- stale presence;
- bots;
- board connections;
- moderator connections;
- shared venue NAT;
- and platform degradation.

Determine whether fairness is:

- constitutional principle;
- state-machine invariant;
- default ordering;
- best-effort behavior;
- commercial limit;
- or marketing language.

Require explicit override behavior and user-visible consequences.

## 10. Reliability, operations, recovery, and solo operation

Test whether one person can actually run the product.

Review:

- health checks;
- diagnostics;
- structured logs;
- alerts;
- counters;
- runbooks;
- emergency stop;
- configuration propagation;
- credential rotation;
- backup and restore;
- rollback;
- database migration;
- partial deployment;
- provider outage;
- DNS/domain failure;
- email failure;
- queue or scheduled-work failure;
- data corruption;
- abuse incident;
- privacy request;
- legal takedown;
- and operator absence.

A `200` health endpoint is not sufficient merely because two storage calls succeeded.

Determine what health proves and what it does not prove.

For every public release gate ask:

- Which operational controls already exist?
- Which arrive later?
- Which incident cannot yet be handled safely?
- Is “beta” being used to excuse preventable harm?
- Is rollback defined?
- Is recovery tested or only described?
- Does a release preserve live events?
- Are manual prerequisites discoverable and auditable?

## 11. Accessibility, language, and hostile content

Review all surfaces and emails for:

- keyboard use;
- focus management;
- screen readers;
- live-region behavior;
- reduced motion;
- contrast;
- zoom;
- projectors;
- small mobile screens;
- intermittent connectivity;
- localization expansion;
- pluralization;
- German Du and formal Sie;
- locale fallback;
- date/time formatting;
- bidirectional text;
- and hostile user-generated strings.

Do not accept a generic WCAG target without concrete behavior for live updates, announcements, line movement, errors, reconnect, and moderation actions.

## 12. Economics, entitlements, and platform dependence

Review:

- free baseline;
- fair-use limits;
- entitlement seam;
- invite codes;
- lockdown;
- future monetization;
- accounts;
- cost brakes;
- Cloudflare quotas;
- public beta;
- operational attention;
- and self-hosting.

Challenge contradictions among:

- all features being free;
- future gating;
- never-gated features;
- invite-only mode;
- vouchers;
- fairness limits;
- accountless participation;
- and monetization being parked.

Determine whether the product remains coherent if monetization never ships.

Determine whether self-hosting is:

- enduring principle;
- release feature;
- repository convenience;
- support promise;
- or unsupported marketing language.

Separate Cloudflare-specific deployment from a general self-hosting promise.

## 13. Constitution suitability

For every proposed principle ask:

- Is it enduring?
- Is it stable across implementation changes?
- Does it govern future trade-offs?
- Is it enforceable?
- Can a spec, plan, task list, pull request, or release gate be checked against it?
- Does it include exceptions?
- Who may invoke those exceptions?
- Does it conflict with another principle?
- Does it belong in product policy, technical context, runbook, or ordinary requirements instead?
- Would changing it require a constitutional amendment?

Reject wording that is:

- aspirational but unenforceable;
- duplicated from a feature requirement;
- tied to a vendor without enduring justification;
- a vague slogan;
- a fabricated legal conclusion;
- or incompatible with another article.

Do not make the opposite error: a genuine enduring constraint must not be demoted because it is difficult.

</mandatory_attack_surfaces>

<gitignore_audit>

Treat `.gitignore` as a security and repository-integrity artifact, not a cosmetic file.

The current repository already knows enough about the planned implementation to perform a serious pre-implementation audit.

Review `.gitignore` against:

- the pinned and scheduled stack in `docs/engineering/tech-context.md`;
- the monorepo layout;
- pnpm workspaces;
- Turborepo;
- Node and TypeScript;
- React Router and Vite;
- Cloudflare Workers and Wrangler;
- local D1, KV, Durable Object, Miniflare, and Local Explorer state;
- Drizzle and migrations;
- Vitest;
- coverage tooling;
- Playwright;
- Lingui catalogs;
- generated build artifacts;
- caches;
- logs;
- editor and OS artifacts;
- local secrets and environment files;
- package-manager artifacts;
- Spec Kit;
- Claude Code;
- and future self-hosting scaffolding.

Build a two-sided matrix:

### Must be ignored

Local secrets, runtime state, caches, reports, temporary artifacts, local databases, generated output, and machine-specific files that must never be tracked.

### Must remain trackable

Lockfiles, manifests, workspace files, `wrangler.jsonc`, migrations, source catalogs, safe example files, CI workflows, specs, documentation, templates, and reproducibility artifacts.

Specifically challenge overly broad patterns such as `.env*` that may also hide safe example files.

Use mechanical checks where possible:

```bash
git status --ignored --short
git check-ignore -v --no-index <path>
```

You may use a newline-delimited representative path list with `git check-ignore --stdin --no-index` so the audit does not require creating fake files.

Requirements:

1. Derive entries from the declared stack and current official tool behavior, not generic copy-paste templates.
2. Do not ignore a path merely because it is inconvenient.
3. Do not hide files that are needed for reproducibility, migrations, source translation, review, or deployment.
4. Add explicit negation rules for safe examples when broad secret patterns would mask them.
5. Group the final file by purpose and comment non-obvious entries.
6. If an expected artifact cannot yet be known until F000 selects or generates the real scaffold, record an implementation-time verification requirement rather than guessing.
7. If the current `.gitignore` is incomplete, amend it during this review.
8. Verify the final patterns against representative paths.
9. Check that `.dev.vars.example` remains trackable and contains keys only.
10. Record the audit result and residual F000 verification items in the relevant documentation.

A constitution-ready repository must not knowingly enter implementation with an obviously incomplete or dangerously broad `.gitignore`.

</gitignore_audit>

<repository_operations_guide>

Create a central, authoritative guide that humans and agents can use for **all repository and CLI interactions**.

Default path:

`docs/runbooks/repository-operations.md`

Use another path only if the existing authority model proves a different location is materially better. Explain the choice.

This guide must not duplicate the build guide. It is the concise operational command authority; the build guide remains the process authority.

Build its inventory mechanically by searching:

- tracked Markdown code fences;
- shell scripts;
- PowerShell scripts;
- GitHub workflows;
- package manifests;
- Spec Kit files;
- CLAUDE.md;
- CONTRIBUTING.md;
- runbooks;
- and configuration.

At minimum cover every current or planned CLI named or used by the repository, including where applicable:

- `git`;
- GitHub CLI `gh`;
- `claude`;
- GitHub Spec Kit commands;
- `fnm`;
- `node`;
- Corepack;
- `pnpm`;
- Turborepo;
- Wrangler;
- Drizzle tooling;
- Vitest;
- Playwright;
- Biome;
- TypeScript;
- gitleaks;
- and repository scripts introduced by F000.

The guide must clearly distinguish:

- commands available now;
- commands planned but not yet created;
- commands whose exact syntax must be reverified at F000 or a later implementation milestone;
- read-only commands;
- local mutating commands;
- remote mutating commands;
- destructive or high-risk commands;
- commands humans may run;
- commands agents may run autonomously;
- and commands requiring explicit operator approval.

Required sections:

1. **Purpose, scope, and authority**
2. **Repository preflight and session start**
3. **Reading current state and `PROGRESS.md`**
4. **Git branch and working-tree policy**
5. **Safe inspection commands**
6. **Creating and switching feature branches**
7. **Synchronizing with `main`**
8. **Reviewing diffs**
9. **Staging policy**
10. **Conventional commits**
11. **Push, pull request, review, merge, and branch cleanup**
12. **Release and tag operations**
13. **Recovery from common Git mistakes**
14. **Explicitly forbidden shortcuts and destructive commands**
15. **GitHub CLI operations**
16. **Node, fnm, Corepack, and pnpm**
17. **Project validation commands**
18. **Wrangler local-development commands**
19. **Wrangler secret handling**
20. **Cloudflare resource and configuration commands**
21. **D1 migrations and why remote migrations run only in CI/CD**
22. **Spec Kit and Claude Code command flow**
23. **Logs, command output, and manual configuration recording**
24. **Environment matrix: local, DEV, PRD**
25. **Command verification and version drift**
26. **Agent permission matrix**
27. **Quick-reference safe command sequence**

The guide must enforce at least these rules:

- inspect status before and after changes;
- never overwrite unknown work;
- never use `git add .` as the default;
- inspect staged diff before committing;
- use conventional commits;
- keep commits small and coherent;
- never commit secrets;
- never place secrets directly in documented command lines;
- never deploy with local `wrangler deploy`;
- never apply remote D1 migrations locally;
- production deploys occur only through the documented GitHub release and protected-environment path;
- remote mutations require explicit operator intent;
- agents do not push, merge, release, delete branches, edit GitHub settings, or mutate Cloudflare by default;
- commands that do not exist yet must be labeled as future/planned rather than presented as executable;
- current official syntax must be reverified at the milestone named by the technical context;
- and the runbook must link to deeper process or configuration documents instead of copying them.

After creating the guide:

- update `docs/README.md` in the same patch;
- add a short pointer from root `CLAUDE.md`;
- reconcile `CONTRIBUTING.md`, the build guide, and existing runbooks;
- eliminate contradictory command advice;
- verify every relative link;
- verify every executable command that is supposed to work now;
- clearly mark commands that cannot yet work because F000 has not created the implementation.

Do not falsely imply that the current documentation-only repository already contains `package.json`, scripts, Wrangler config, or application commands if it does not.

</repository_operations_guide>

<repository_specific_hypotheses>

Test every hypothesis below against the actual local repository.

These are not predetermined defects. Mark each:

- `CONFIRMED`
- `PARTIALLY-CONFIRMED`
- `REJECTED-WITH-EVIDENCE`
- `BLOCKED-BY-MISSING-INFORMATION`

1. Privacy language may undercount names, free text, cookies, operational metadata, email, exports, recovery data, logs, counters, and processor copies.
2. Fixed deletion promises may conflict with point-in-time recovery, exports, email-provider retention, emergency snapshots, audit evidence, and self-hosting.
3. Legal and privacy wording may be stronger than the documented behavior can guarantee.
4. “Committed events are sacred” or fail-open behavior may conflict with emergency stop, credential compromise, abuse intervention, legal takedown, privacy deletion, data corruption, or provider failure.
5. Administrative operational needs may precede the milestone that delivers the complete admin UI.
6. The dual-mode admin wall may leave configuration, rotation, revocation, recovery, and dependency-failure behavior unclear.
7. The health endpoint may be truthful about only a narrow slice of service health.
8. Scheduling, create-ahead, rescheduling, auto-start, auto-end, timezone, and DST rules may not form one deterministic model.
9. Concurrent-presence capacity may be underspecified for reconnect, stale sessions, multiple tabs, mobile suspension, and read-only viewers.
10. Retry and idempotency rules may not cover every participant and moderator command.
11. The prohibition on durable moderator-action trails may conflict with incident response, abuse investigation, diagnostics, or state reconstruction.
12. “No tracking” and “no cookie banner” may depend on future telemetry, geolocation, operational signals, and legal interpretation.
13. Email-locked vouchers may create personal-data correction, deletion, reuse, support, and retention obligations not fully represented.
14. Country data may be described as aggregate-only while still existing on an identifiable account or event record.
15. Account inactivity deletion may conflict with future events, wallets, grants, exports, pending emails, or legal records.
16. The entitlement seam may be treated as foundational while full entitlement behavior arrives much later.
17. The public-beta gate may precede controls required to operate, diagnose, restrict, or recover the service safely.
18. “All features free” and later monetization may lack a stable boundary.
19. `v1.0.0` or “production-mature” language may conflict with parked economics, scheduled legal verification, load testing, degraded-mode decisions, or missing operational evidence.
20. Self-hosting may be promised without clear topology, upgrades, migrations, security ownership, domains, email, backups, support, and compatibility boundaries.
21. Product feature IDs and F000–F013 work-package IDs may drift or be confused.
22. Earlier reviews may have amended conclusions without updating every dependant.
23. Operator notes may have become decisions before their downstream consequences were fully analyzed.
24. Constitution-input articles may contain principles that cannot all be honored during a severe incident.
25. Some “spec-time” or “implementation-time” decisions may actually affect product law and therefore block the constitution.
26. The current `.gitignore` may be too small for the planned stack or too broad around environment examples.
27. Existing Git and CLI instructions may be scattered, contradictory, stale, unsafe for agents, or presented before the commands actually exist.
28. The repository may lack one operational authority that clearly tells humans and agents what they may execute, what they must verify, and what requires operator approval.
29. Previous review language may have declared readiness without a true cold self-review of the resulting diff and full document graph.
30. `PROGRESS.md`, the repository state, and the actual pre-constitution gate may not agree.

Do not omit a hypothesis from the final disposition table.

</repository_specific_hypotheses>

<external_fact_protocol>

Use current external sources only when a repository claim depends on a changing fact, including:

- Claude Code;
- Claude Fable 5;
- GitHub and GitHub CLI;
- Spec Kit;
- Cloudflare products and Wrangler;
- pnpm, Node, React Router, Vite, Turborepo, Drizzle, Vitest, Playwright, Biome, Lingui, or gitleaks;
- browser behavior;
- accessibility standards;
- licensing;
- or law.

For technical facts:

- prefer official primary documentation;
- record retrieval date;
- identify relevant version, plan, region, runtime, or account assumptions;
- distinguish documented guarantee from inference;
- and add implementation-time reverification where appropriate.

For Cloudflare, follow the repository’s current Cloudflare documentation protocol.

For legal matters:

- do not act as counsel;
- identify the exact legal claim;
- state the product behavior that can be committed independently of legal interpretation;
- require professional verification when material;
- prevent unverified legal conclusions from becoming constitutional facts.

If verification is unavailable, mark the statement `UNVERIFIED`.

Do not fill the gap from memory.

</external_fact_protocol>

<questioning_protocol>

After the initial repository sweep, present:

1. `PROVISIONAL VERDICT: NOT READY FOR PHASE 3`
2. baseline;
3. corpus coverage;
4. hypothesis status so far;
5. highest-severity findings;
6. Question Batch 1.

Ask no more than **five questions per batch**.

Use an unlimited number of batches.

Prioritize:

1. constitution-blocking contradictions;
2. user rights, data, security, abuse, and irreversible state;
3. core product and lifecycle ambiguity;
4. milestone and release infeasibility;
5. operations and recovery;
6. repository-governance and CLI safety;
7. traceability;
8. lower-severity defects.

Each question must use this structure:

### Q-### — [finding ID] Exact decision title

**Evidence**  
Exact paths and headings or line ranges.

**Current conflict or gap**  
What is contradictory, absent, unsafe, or unsupported.

**Why it matters**  
Concrete consequence.

**Hostile scenario**  
One realistic scenario demonstrating the defect.

**Reviewer recommendation**  
Your preferred answer and rationale.

**Credible alternatives**  
Only viable alternatives and their trade-offs.

**Exact decision required**  
One decision, not “please clarify”.

**Required answer shape**  
The fields the operator must supply, such as state transition, exception, retention anchor, owner, trigger, guarantee, or risk acceptance.

Do not ask questions already answered unambiguously by the correct authority.

Do not ask the operator to perform repository research you can do yourself.

Do not bundle unrelated decisions.

After each answer:

1. restate the decision precisely;
2. identify assumptions;
3. test it against documents and a hostile scenario;
4. identify downstream changes;
5. state whether the answer is sufficient;
6. ask a focused follow-up when necessary;
7. update the ledger.

Do not ask “Should I continue?”

Continue automatically after the operator responds.

</questioning_protocol>

<weak_answer_rules>

### “Use your recommendation”

Restate the exact recommendation, material consequences, affected documents, and residual risk. Do not record a vague consent.

### “Later”

Require:

- why safe to defer;
- owner;
- milestone or trigger;
- required evidence;
- destination artifact;
- fallback;
- and proof that it does not affect the constitution.

### “Best effort”

Require the minimum observable guarantee, failure behavior, detection, and recovery path.

### Risk acceptance

Record:

- exact risk;
- affected users;
- worst credible impact;
- probability basis if known;
- mitigation;
- detection;
- recovery;
- review trigger;
- and why the risk does not block Phase 3.

### Finding rejection

Require evidence or product rationale, then mark one of:

- `REJECTED-AS-NOT-A-DEFECT`
- unresolved contradiction
- severity escalation

### Technically impossible answer

Say so directly. Identify incompatible constraints and force an explicit trade-off.

### Contradiction with an earlier answer

Present both decisions, affected artifacts, and consequences. Require explicit supersession.

</weak_answer_rules>

<editing_protocol>

Reconnaissance is read-only.

Do not edit a disputed decision before the operator resolves it.

Once a coherent group of decisions is accepted:

1. identify the authoritative owner;
2. identify every dependant;
3. state the amendment set;
4. apply small, coherent patches;
5. preserve historical rationale;
6. retain stable IDs;
7. mark superseded material;
8. avoid unnecessary renumbering;
9. update links, indexes, and maps;
10. update `docs/README.md` in the same patch when adding, moving, or retiring a document;
11. update `PROGRESS.md` only when workflow state genuinely changes;
12. immediately verify the patch.

Do not:

- rewrite provenance to make history look cleaner;
- erase evidence that a decision changed;
- hide disagreement behind abstraction;
- resolve a product defect with an implementation guess;
- turn a current vendor mechanism into a constitutional principle;
- or demote a constitutional constraint to avoid its consequence.

Typical propagation may include:

- product brief;
- feature inventory;
- decision log;
- constitution input;
- tech context;
- feature briefs;
- build-guide gates;
- runbooks;
- `.gitignore`;
- documentation map;
- CLAUDE.md;
- CONTRIBUTING.md;
- and PROGRESS.md.

After each editing wave:

```bash
git status --short
git diff --check
git diff --stat
git diff
```

Then:

- search for obsolete wording;
- search every changed ID;
- check relative Markdown links;
- check heading anchors where referenced;
- validate command snippets that are claimed to work now;
- validate `.gitignore` representative paths;
- and rerun the affected cross-document checks.

</editing_protocol>

<traceability_checks>

Create mechanical or semi-mechanical checks where practical.

At minimum verify:

- Markdown relative links resolve;
- referenced headings exist;
- active feature IDs are unique;
- feature-brief inventory IDs exist;
- every M1–M4 inventory item maps exactly once to F000–F013;
- every expected work-package brief exists;
- milestone and version references agree with the inventory;
- decision IDs are unique;
- referenced decisions exist;
- superseded decisions point to successors;
- parked and rejected work is not active;
- constitution-input concepts have product and decision evidence;
- data-lifecycle claims agree;
- current-position claims agree with `PROGRESS.md`;
- provenance-only documents are not treated as authority;
- root CLAUDE.md points to the right operational sources;
- repository command guidance is not contradictory;
- commands marked “available now” actually exist;
- future commands are labeled honestly;
- `.gitignore` does not conceal required reproducibility files;
- and newly added documentation appears in `docs/README.md`.

Do not claim complete traceability from a sample.

Report the method and limitations.

</traceability_checks>

<mandatory_self_review>

After you believe all accepted changes are complete, do not proceed directly to a verdict.

Perform a rigorous self-review of your own work.

This is mandatory even if independent subagents were already used.

## Pass A — Intent and completeness review

Re-read:

- the operator’s original objective;
- every operator answer;
- the finding ledger;
- every changed file;
- and the full final diff.

Verify that every accepted decision was implemented exactly once in the correct authority and propagated to all dependants.

## Pass B — Hostile regression review

Assume your changes are wrong.

Try to find:

- new contradictions;
- scope accidentally added or removed;
- stale references;
- broken authority boundaries;
- duplicate current answers;
- weakened security or privacy wording;
- stronger legal claims than before;
- milestone drift;
- command advice that can damage the repo or mutate remote systems;
- secrets or example values;
- `.gitignore` rules that hide required files;
- commands presented as working before F000 creates them;
- and review conclusions unsupported by evidence.

## Pass C — Mechanical integrity review

Run and inspect:

- `git status --short`
- `git diff --check`
- complete `git diff`
- Markdown link checks
- heading-reference checks
- identifier checks
- traceability checks
- `.gitignore` representative-path checks
- command-existence checks
- and any repository-provided validation available at this documentation-only stage.

## Pass D — Fix and repeat

Fix every issue found in Passes A–C.

Then repeat the affected passes.

Do not merely list self-review defects in the final report. Correct them unless a product decision is required, in which case reopen the finding and ask the operator.

The self-review is complete only when a fresh pass finds no unresolved issue.

</mandatory_self_review>

<fresh_independent_verification>

After the self-review passes, launch a fresh read-only verifier with minimal exposure to your prior conclusions.

The verifier must receive:

- the current repository;
- the readiness criteria;
- the task of disproving readiness;
- and no summary that biases it toward your conclusions.

Ask it to:

1. reconstruct the authority map independently;
2. identify the actual workflow position;
3. find contradictions across product, engineering, build-guide, runbook, `.gitignore`, CLAUDE.md, and progress artifacts;
4. test all repository-specific hypotheses;
5. review the repository-operations guide for safety and truthfulness;
6. review `.gitignore` for both missing and overbroad patterns;
7. inspect the final diff for regressions;
8. propose at least ten plausible reasons the repository might still be unready;
9. reject unsupported reasons;
10. return an independent verdict with evidence.

You must investigate every verifier finding.

Do not dismiss a finding because your earlier passes missed it.

Fix valid findings, repeat self-review, and rerun independent verification when material changes were required.

A self-review alone is not sufficient.

An independent review alone is not sufficient.

Both must pass.

</fresh_independent_verification>

<operator_readiness_exam>

Repository consistency is necessary but not sufficient.

Before a READY verdict, conduct a closed-book teach-back in short rounds.

Do not provide the answer inside the question.

The operator must be able to explain and apply:

1. product problem, design center, supported contexts, and non-goals;
2. every actor and trust boundary;
3. event, question, participant, moderator, account, voucher, entitlement, admin, export, purge, and emergency lifecycles;
4. timing, timezone, reconnect, retry, concurrency, and idempotency rules;
5. data categories, retention, deletion domain, exports, logs, counters, and recovery copies;
6. privacy claims and their limits;
7. abuse controls and residual risks;
8. what “the line is sacred” means;
9. what “committed events are sacred” means and its exceptions;
10. capacity and fairness;
11. public-beta operational risk;
12. single-operator incident and recovery posture;
13. milestone and SemVer model;
14. feature IDs versus F000–F013 work packages versus Spec Kit numbers;
15. open, scheduled, assumed, parked, rejected, amended, and superseded decisions;
16. why every remaining scheduled decision is safe to defer;
17. candidate constitutional principles and tensions;
18. which mechanisms must stay out of the constitution;
19. `.gitignore` safety model;
20. repository Git and CLI safety policy;
21. which commands an agent may run autonomously;
22. which commands require explicit operator approval;
23. why local deploys and remote migrations are prohibited;
24. and the exact gate for beginning Phase 3.

For an incomplete or incorrect answer:

- identify the gap;
- return to the relevant artifact;
- correct documentation if necessary;
- and test the operator later with a different scenario.

Do not award readiness for memorized wording.

Require applied understanding.

</operator_readiness_exam>

<readiness_gate>

You may issue `READY FOR PHASE 3 — CONSTITUTION` only when all are true:

- the baseline is recorded;
- the mandatory corpus is reviewed;
- every repository-specific hypothesis has a disposition;
- no blocker is unresolved;
- no high-severity finding is unresolved;
- no answer is `ANSWER-INCOMPLETE`;
- every medium finding is fixed, legitimately deferred, rejected with evidence, or covered by explicit risk acceptance;
- every constitution-material assumption is decided or represented honestly with a safe resolution path;
- product behavior and technical mechanisms are separated correctly;
- product, state, data, security, abuse, and operational lifecycles are coherent;
- feature, milestone, SemVer, and work-package mappings agree;
- parked and rejected work has not leaked into active scope;
- external facts are verified or explicitly marked for reverification;
- legal conclusions have not been fabricated;
- constitution inputs are enduring, enforceable, non-duplicative, and mutually compatible;
- `.gitignore` has been audited, amended where needed, and mechanically checked;
- the central repository-operations guide exists, is mapped, is safe for humans and agents, and does not pretend future commands already exist;
- every required document amendment is applied;
- links, headings, IDs, and command references pass;
- the final diff contains no accidental changes;
- the mandatory self-review passes;
- the fresh independent verifier finds no unresolved blocking issue;
- the operator passes the teach-back;
- remaining scheduled decisions are proven not to alter constitutional meaning;
- and `PROGRESS.md` truthfully represents the state.

Prior approval satisfies none of these criteria by itself.

Document volume satisfies none of these criteria.

Model confidence satisfies none of these criteria.

When evidence is incomplete, the verdict remains `NOT READY FOR PHASE 3`.

</readiness_gate>

<final_report>

When the gate passes, provide:

# PRE-CONSTITUTION HOSTILE REVIEW — FINAL REPORT

## 1. Verdict

Exactly:

`READY FOR PHASE 3 — CONSTITUTION`

or:

`NOT READY FOR PHASE 3`

## 2. Baseline

- repository root
- branch
- baseline commit
- starting worktree state
- review dates
- corpus coverage

## 3. Findings summary

Counts by severity and closure status.

## 4. Hypothesis disposition

All repository-specific hypotheses, with evidence.

## 5. Material corrections

For each:

- finding ID
- original defect
- accepted decision
- changed files
- verification

## 6. Reopened, amended, or superseded decisions

Include predecessor and successor references.

## 7. Risk acceptances

Include impact, mitigation, detection, recovery, and review trigger.

## 8. Legitimately remaining open or scheduled decisions

For each:

- reason it does not block the constitution
- owner
- trigger
- required evidence
- destination artifact
- fallback

## 9. Cross-document integrity

Report:

- authority consistency
- product lifecycle consistency
- data lifecycle consistency
- state-machine consistency
- feature mapping
- milestone and version mapping
- work-package mapping
- link and identifier checks

## 10. `.gitignore` audit

Report:

- missing rules corrected
- overbroad rules corrected
- must-track files protected
- representative-path test results
- F000 reverification items

## 11. Repository and CLI operations guide

Report:

- path
- authority
- tools covered
- commands verified now
- commands marked future
- human/agent permission model
- reconciled documents

## 12. Constitution-preparation map

For each candidate principle:

- name
- product evidence
- decision evidence
- intended constitutional force
- enforceable consequence
- exception
- tension with other principles
- implementation mechanisms explicitly excluded

Do not draft the constitution.

## 13. Self-review result

Report each mandatory pass, defects found, fixes applied, and final rerun.

## 14. Independent-verifier result

List findings and dispositions.

## 15. Operator-readiness result

State demonstrated strengths and remaining knowledge gaps without flattery.

## 16. Changed files

Every changed file and purpose.

## 17. Final worktree state

Show:

```bash
git status --short
```

Do not commit.

## 18. Exact next step

Point to:

`docs/build-guide/04-product-definition.md` → `§5 Phase 3 — Constitution`

Do not execute it.

If the verdict is NOT READY, instead end with:

- unresolved blockers;
- exact decisions required;
- inconsistent artifacts;
- next question batch;
- and no Phase 3 instruction.

</final_report>

<behavioral_constraints>

Throughout this review:

- lead with evidence;
- cite exact paths and headings;
- separate fact, inference, recommendation, decision, and risk acceptance;
- inspect before asking;
- ask precise questions;
- never invent file contents;
- never claim a command passed without running it;
- never hide uncertainty;
- never soften a material defect;
- never use verbosity as a substitute for precision;
- never reveal private chain-of-thought; provide findings, evidence, trade-offs, and conclusions;
- never ask the operator to repeat repository facts;
- never stop at the first plausible resolution;
- never ask whether to continue;
- never treat context pressure as completion;
- never move to the constitution with one blocking contradiction;
- never leave a valid self-review finding unfixed;
- never let the operator’s approval override contradictory evidence without an explicit product trade-off;
- and never allow a dangerous command, secret-handling pattern, or misleading operational instruction to become repository authority.

</behavioral_constraints>

<start_now>

Begin now.

1. Establish the Git baseline.
2. Read the documentation map and progress tracker.
3. Read the Phase 3 handoff.
4. Inventory the entire tracked corpus.
5. Inspect pre-existing changes.
6. Run the independent hostile review passes.
7. Audit `.gitignore`.
8. Inventory Git and CLI instructions.
9. Populate the temporary ledger.
10. Present the provisional verdict, corpus coverage, highest-severity findings, and Question Batch 1.

Do not ask an onboarding question before the initial repository sweep.

Assume nothing is constitution-ready until evidence proves it.

</start_now>
```

**🛑 GATE 2:** skim `tech-context.md` end-to-end — this file steers every plan; wrong here = wrong everywhere. Confirm the work-package cut (it becomes the F-numbers in [file 06](06-m1-to-m4-plan.md) and PROGRESS.md — update both if it changed). Commit: `docs: tech context + feature briefs`.

From this gate on, **`docs/inputs/tech-decisions.md` is provenance, not authority**: it stays in the repo as the record of what was decided before Spec Kit took over, but plans and prompts cite only `tech-context.md` (file 06's plan addenda do exactly that). If reality changes later, update tech-context.md and the decision log — never the input file. This keeps the Spec Kit separation clean: the WHAT lives in specs, the HOW lives in one living document, and pre-history lives in `docs/inputs/`.

---

## §5 Phase 3 — Constitution

The constitution is Spec Kit's enforcement mechanism: every later `/speckit.plan` and `/speckit.analyze` checks itself against it. Write it once, carefully.

```text
PROMPT P3 — constitution
[MODEL: Fable 5] [EFFORT: max] [SESSION: fresh]

/speckit.constitution Establish the MicLine.app constitution from
docs/product/constitution-input.md and docs/product/product-brief.md. It must
contain at least these articles (tighten wording, add rationale, flag anything
you believe is missing or self-contradictory before finalizing — and check the
product brief's "Product principles" section: any principle that must bind
implementation, e.g. "the line is sacred", "committed events are sacred",
zero-gate participation, races-resolve-into-states, deserves an article or an
explicit home in one):

I.   PUBLIC REPOSITORY, ZERO SECRETS. No secret, credential, token, or personal
     data in any tracked file, commit, log statement, or test fixture — ever.
     Secrets exist only in .dev.vars (gitignored), Wrangler secrets, and GitHub
     Actions secrets. `.dev.vars.example` documents keys with NO values.
II.  CLOUDFLARE-FIRST. All compute, storage, delivery, email, and bot defense
     run on Cloudflare. External third-party APIs are permitted ONLY where no
     Cloudflare primitive exists (e.g. future payments), must sit behind a
     service-layer seam, and require a decision-log entry.
III. BUILT TO RUN FOR YEARS BY ONE PERSON. Prefer boring, first-party, managed
     primitives; minimize dependencies; no component that needs babysitting.
     Every feature plan must answer: "what does this cost the operator per
     month in money and attention?"
IV.  REALTIME CORRECTNESS. The EventRoom Durable Object is the single authority
     for live event state. Invariants (one "Now speaking" max; one active
     waiting item per participant in the open line; the line is never silently
     reordered; position stability in open mode; skip-stub visibility rules;
     terminal-state immutability; vote uniqueness once M9 upvotes exist) are
     enforced in the DO and covered by tests. Clients render server state;
     they never compute authoritative state.
V.   PRIVACY BY DEFAULT (operator is in Germany; GDPR applies). Participants
     require no account and no PII — nothing to delete beyond an anonymous
     session cookie; moderator data is email + optional display name + country
     (request geolocation, anonymous aggregates only) + console language, and
     nothing else. Every stored datum has a defined lifetime and a deletion
     path (the product brief's data-lifecycle table is binding); anonymous
     aggregates survive deletion only if unlinkable by construction. No
     third-party analytics/trackers on any surface. No fingerprinting or
     IP-identity binding, ever.
VI.  ONE ENTITLEMENT SEAM. Anything gated (features, event counts, limits) is
     checked exclusively through a single capability interface in the service
     layer. Grants come from the free baseline, invite codes (M8), and admin
     action, and MAY come from billing later; gated code never knows the
     difference. The seam is anticipated architecturally from M1 even though
     the entitlement system ships in M8.
VII. QUALITY GATES. TypeScript strict; zod validation at every trust boundary;
     domain logic (state machine, auth, entitlements) has unit tests covering
     every transition/branch; CI (lint, typecheck, test, build) must be green
     to merge; main is protected.
VIII.SPEC-DRIVEN CHANGE. No feature code without spec → plan → tasks under
     specs/. Drift is corrected by amending artifacts (or /speckit.converge),
     never by silently diverging code.
IX.  ACCESSIBLE, MOBILE-FIRST, MULTILINGUAL SURFACES. The participant
     experience is designed for phones in a dark event hall: readable,
     thumb-reachable, WCAG AA, functional on flaky venue Wi-Fi (reconnect
     gracefully). Every user-facing string flows through the i18n layer from
     the first commit — adding a locale is a translation task, never a
     refactor.
```

**🛑 GATE 3:** read `.specify/memory/constitution.md`. This is your veto document — every word must be something you're willing to enforce for years. Edit freely, then commit: `docs: constitution v1`.

---

## §6 Phase 4 — Epic briefs (M5–M13)

Per our agreed depth gradient: the M1–M4 arc gets full Spec Kit rigor now (§7–§8 — its briefs came out of Gate 2); M5–M13 get **epic briefs** whose only job is to (a) park the ideas in a gated, findable place and (b) force forward-compatible decisions into M1–M4 now. What this phase deliberately does **not** do anymore: challenge the milestone cut — that happened on the product side (decision log §P3, 2026-07-12), and the approved M1–M13 lineup with its target versions lives in `docs/product/feature-inventory.md`, which stays the canonical milestone map (no separate `milestone-map.md` — one authority, zero drift). If the tech context surfaced a *structural* problem with the lineup, that's a product-decision reopen in the decision log, not a quiet re-cut here.

```text
PROMPT P4 — epic briefs M5–M13
[MODEL: Fable 5] [EFFORT: high] [SESSION: fresh]

Read docs/product/feature-inventory.md, docs/product/decision-log.md (P3 +
FR sections + the "Scheduled decisions" registry), docs/engineering/
tech-context.md, and the constitution. The M1–M13 lineup and all milestone
ownership are DECIDED (decision log §P3) — do not re-cut; if the tech
context genuinely contradicts the lineup somewhere, flag it as a proposed
product-decision reopen and stop for my call.

1. Write docs/product/epics/ — one file per post-beta milestone:
     M5-moderator-control-and-data-portability.md · M6-operator-control-
     plane.md · M7-emergency-operations-and-recovery.md · M8-entitlements-
     invite-codes-and-lockdown.md · M9-curated-moderation.md · M10-extended-
     event-toolkit.md · M11-design-system-identity-and-experience-overhaul.md
     · M12-operational-maturity-and-compliance.md · M13-monetization-if-ever.md
   Each ≤ 2 pages: goal (the inventory's release outcome, expanded), feature
   list (the milestone's inventory IDs), NON-goals, the kickoff-owned
   decisions from the decision log's "Scheduled decisions" registry, and —
   most important — a "Demands on M1–M4" section listing every concrete
   forward-compat requirement the current build arc must satisfy, e.g.:
   - M6/M8 → M1: `requireAdmin()` as the wall-guard seam (AF-7: the wall is
     the admin auth — no `users.role` column, the admin is not a product
     user); all gateable behavior behind the Article-VI entitlement interface
     with a hardcoded free baseline; append-only audit-log table for
     privileged admin/entitlement actions only; event/user soft-delete
     semantics; ops-counter writes from each signal's milestone (AF-3).
   - M5/M7 → M3/M4: the recap/export data must be assemblable from what the
     DO + D1 actually store; the M7 emergency machinery consumes the M5
     export service — design the M4 recap with that seam in mind.
   - M9 → M3: the two-mode-shaped domain model (decision B10) — pool/review/
     declined/merged states and curated ordering must slot in without a
     schema rewrite.
   - M11 → M1–M4: design tokens centralized in packages/ui from the start;
     no inline one-off styling; components separated from data hooks so
     reskinning doesn't touch logic.
   - M12 → M1/M4: structured logs from day one; retention enforcement ships
     in M4 (P3-6) and must be built verifiable — M12 only verifies, monitors,
     and load-tests it.
   - M13 → M8: billing = just another entitlement grantor; no price/plan
     concepts anywhere else. Include the PARKED checklist of German-operator
     duties to resolve with a tax advisor/lawyer before charging anyone
     (Impressum/DDG, USt/Kleinunternehmerregelung, AGB, Widerrufsbelehrung,
     invoicing) — list, don't advise.

2. Cross-check: every "Demands on M1–M4" item must be traceable into an
   M1–M4 feature brief (Gate 2 set) — list any demand that is NOT yet
   covered so I can have the briefs patched.

3. If anything here changes feature statuses, update feature-inventory.md
   accordingly (it stays the single milestone authority). Print the epic
   list and the uncovered-demands list.
```

After the gate, run the GitHub milestone/label setup block from [file 03 §A](03-github-operations.md) — the approved M1–M13 titles — so they exist as GitHub Milestones before the first feature loop.

**🛑 GATE 4:** the "Demands on M1–M4" lists are the whole point — verify each demand actually appears in the relevant M1–M4 feature brief (tell Claude to patch the briefs where missing). Commit: `docs: M5–M13 epic briefs`.

---


**Next:** read [05-feature-loop.md](05-feature-loop.md), then start F000 in [06-m1-to-m4-plan.md](06-m1-to-m4-plan.md). **Carry forward:** everything downstream obeys these artifacts — fix wrongness here, never later.
---
[◀ GitHub operations: environments, releases, lifecycle](03-github-operations.md) · [⌂ Index](README.md) · [The feature loop ▶](05-feature-loop.md)
