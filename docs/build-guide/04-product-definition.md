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
   - M6/M8 → M1: `users.role` column + `requireAdmin()` gate from day one;
     all gateable behavior behind the Article-VI entitlement interface with
     a hardcoded free baseline; append-only audit-log table for privileged
     admin/entitlement actions only; event/user soft-delete semantics.
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
