> **MicLine Build Guide — Part 4 of 10** · [◀ GitHub operations: environments, releases, lifecycle](03-github-operations.md) · [⌂ Index](README.md) · [The feature loop ▶](05-feature-loop.md)

**In this part:** Four prompts (P1–P4) turn "MicLine, roughly" into a product brief, tech context, constitution, and milestone map + epic briefs — with gates 1–4 as your steering wheel.
**Prerequisites:** File 02 complete · file 03 read · ideally the Fable 5 window still open (file 01). · **Your time:** ~2–4 h of your attention spread over 1–3 days (the P1 interview dominates). *(your active attention; Claude runtime and limit pauses excluded)*

## §3 Phase 1 — Product brainstorm (the interview you asked for)

Before the prompts, the mental model you asked for — **how each building block goes from idea to running code**. Nothing is built ad hoc; every block passes the same three stations: *envisioned* (the P1 interview), *pinned* (P2 tech context, P3 constitution, P4 briefs), *built* (the file-05 loop on a numbered feature):

| Building block | Envisioned in | Pinned in | Built as |
|---|---|---|---|
| Landing page / marketing | P1 topics B + F | F007 brief | F007 loop (made *beautiful* in M4) |
| Moderator console & participant UI | P1 topic B | F004/F005 briefs + `packages/ui` tokens | F004/F005 loops (functional in M1 — the deliberate beauty pass is M4, so you never fight design battles twice) |
| Backend & realtime core | P1 topics B + E | tech-context + constitution Art. IV + F001–F003 briefs | F001–F003 loops |
| Admin interface | P1 topic D | M2 epic ("Demands on M1" reach back into F001) | M2 loops |
| Vouchers / entitlements | P1 topic E | constitution Art. VI + M2 epic | M2 loops |
| Locales / i18n | P1 topic H | tech-context + constitution Art. IX | F000 plumbing, then every UI feature |

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

**🛑 GATE 1:** read all three files. Fix wrong assumptions by telling Claude in the same session ("Change decision #12: …; update the files"). Commit: `docs: product definition v1`.

---

## §4 Phase 2 — Establish the tech context

Spec Kit separates WHAT (specs) from HOW (plans). This phase creates the single HOW document every future `/speckit.plan` reads. Your input is `docs/inputs/tech-decisions.md` (ships with this guide — the distilled, pre-verified stack decisions), optionally plus `docs/inputs/legacy-spec.md` if you kept the original planning doc.

```text
PROMPT P2 — establish tech context
[MODEL: Fable 5] [EFFORT: high] [SESSION: fresh]

Read docs/inputs/tech-decisions.md, docs/product/product-brief.md,
docs/product/feature-inventory.md, and — if present — docs/inputs/
legacy-spec.md. The product docs OVERRIDE the technical inputs wherever they
conflict; list every conflict you find in a "Superseded" section at the end.

Then, for each Cloudflare product in the stack (workers, d1, durable-objects,
kv, email-service, turnstile), fetch developers.cloudflare.com/⟨product⟩/
llms.txt and skim the most relevant pages; reconcile what you read against
tech-decisions.md and note every correction.

Produce exactly three artifacts:

1. docs/product/constitution-input.md — enduring PRINCIPLES and CONSTRAINTS
   only (public-repo/zero-secrets; Cloudflare-first; correctness invariants of
   the question-line state machine; testing gates; privacy/data-lifecycle).
   No tech choices, no schemas.

2. docs/engineering/tech-context.md — the curated HOW knowledge every future
   /speckit.plan must read: pinned stack table; architecture (single Worker,
   Hono for /api+/ws, React Router SSR fallthrough, shared service layer so
   loaders and API routes call the same functions); data-model draft marked
   "draft — authoritative version emerges per-feature"; realtime design
   (DO-per-event, WebSocket Hibernation); the question-line state machine;
   auth design (magic links, HMAC cookies); abuse stack; monorepo layout;
   secrets matrix; environment model (DEV/PRD — see docs/build-guide/
   03-github-operations.md); CI/CD; test strategy. Fold the "Architecture
   rules", abuse/hardening defaults, and "Verified findings" sections of
   tech-decisions.md into this file — corrected per your doc check — so
   that NO later plan ever needs to open tech-decisions.md again. End with a
   "Verify at implementation time" list for every external fact.

3. docs/product/feature-briefs/ — one file per M1 feature, canonical set:
     F000 foundations & delivery pipeline · F001 event lifecycle
     F002 moderator auth (magic links) · F003 EventRoom realtime + question line
     F004 moderator console UI · F005 participant experience
     F006 abuse & content safety · F007 marketing site
   Reconcile against the M1 rows of the feature inventory; flag misfits.
   Each brief ≤ 1 page: user value, in-scope, out-of-scope, dependencies —
   WHAT/WHY only; any HOW you're tempted to write goes into tech-context.md.

Print the Superseded list and the file tree you created.
```

**🛑 GATE 2:** skim `tech-context.md` end-to-end — this file steers every plan; wrong here = wrong everywhere. Commit: `docs: tech context + feature briefs`.

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
you believe is missing or self-contradictory before finalizing):

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
     for live event state. Invariants (one `speaking` max; linePosition ⟺
     status=queued; vote uniqueness; terminal-state immutability) are enforced
     in the DO and covered by tests. Clients render server state; they never
     compute authoritative state.
V.   PRIVACY BY DEFAULT (operator is in Germany; GDPR applies). Participants
     require no account and no PII; moderator data is email + optional display
     name only. Every stored datum has a defined lifetime and a deletion path.
     No third-party analytics/trackers on any surface.
VI.  ONE ENTITLEMENT SEAM. Anything gated (features, event counts, limits) is
     checked exclusively through a single capability interface in the service
     layer. Grants come from vouchers today and MAY come from billing later;
     gated code never knows the difference.
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

## §6 Phase 4 — Milestone map + epic briefs (M2–M5)

Per our agreed depth gradient: M1 gets full Spec Kit rigor (§7–§8); M2–M5 get **epic briefs** whose only job is to (a) park the ideas and (b) force forward-compatible decisions into M1 now. One thing to hold clearly: the M1–M5 set used throughout this guide is a **working proposal** — this phase is where the milestone cut itself gets challenged and *defined*, not merely written down. After Gate 4, `milestone-map.md` is canonical and every M-reference in the guide maps onto whatever you fixed here (README, "Conventions").

```text
PROMPT P4 — milestone map + epic briefs
[MODEL: Fable 5] [EFFORT: high] [SESSION: fresh]

Read docs/product/feature-inventory.md, decision-log.md, and the constitution.

0. Before writing anything: interview me on the milestone cut itself (P1
   rules — batches ≤5, be critical). M1–M5 below are my working proposal,
   not a decision: challenge the boundaries, propose merges, splits,
   renames, or additional milestones where the feature inventory suggests
   them, and confirm the final set with me before proceeding. Everything
   below then applies to the CONFIRMED set.

1. Write docs/product/milestone-map.md:
   - M1 "Core product": ordered feature list F000–F007 (I will provide the
     canonical order — see below), each linking its feature brief. Definition
     of done = the M1 release gate.
   - M2 "Stand-out + operations": admin interface, voucher/entitlement system,
     and the force-ranked keeper features from the brainstorm.
   - M3 "Monetization (PARKED)": explicitly parked; brief only. State in the
     map that EXECUTION order is M1 → M2 → M4 → M5 → M3-if-ever: MicLine runs
     fully live, polished, and operable before billing is even reconsidered.
   - M4 "UI overhaul": the four surfaces (landing, moderator console, audience
     board, admin) using the design direction captured in the brainstorm.
   - M5 "Longevity": performance, cost, stability, ops.
   Canonical M1 order: F000 foundations/CI, F001 event lifecycle (create/share/
   end, no auth-gating yet beyond stub), F002 moderator auth (magic links),
   F003 EventRoom realtime + question-line domain, F004 moderator console UI,
   F005 participant experience UI, F006 abuse & content safety, F007 marketing
   site. If the brainstorm changed M1 scope, adjust and tell me.

2. Write docs/product/epics/M2-admin-and-entitlements.md, M3-monetization.md,
   M4-ui-overhaul.md, M5-longevity.md. Each ≤ 2 pages: goal, feature list,
   NON-goals, and — most important — a "Demands on M1" section listing every
   concrete forward-compat requirement M1 must satisfy, e.g.:
   - M2 → M1: `users.role` column ('user'|'admin') + `requireAdmin()` gate from
     day one; all gated behavior behind the Article-VI entitlement interface
     with a hardcoded free baseline; append-only audit-log table for privileged
     actions; event/user soft-delete semantics.
   - M3 → M1/M2: billing = just another entitlement grantor; no price/plan
     concepts anywhere else. Include a PARKED checklist of German-operator
     duties to resolve with a tax advisor/lawyer before launch (Impressum/DDG,
     USt/Kleinunternehmerregelung, AGB, Widerrufsbelehrung, invoicing) — list,
     don't advise.
   - M4 → M1: design tokens centralized in packages/ui from the start; no
     inline one-off styling; components separated from data hooks so reskinning
     doesn't touch logic.
   - M5 → M1: structured logs from day one; event end-of-life policy (DO data
     retention/cleanup) DECIDED now even if enforcement ships in M5 — propose
     options and ask me to pick.

3. Update feature-inventory.md statuses/milestones to match. Print the map.
```

After the gate, run the GitHub milestone/label setup block from [file 03 §A](03-github-operations.md) — using the milestone titles Gate 4 confirmed — so they exist as GitHub Milestones before the first feature loop.

**🛑 GATE 4:** the "Demands on M1" lists are the whole point — verify each demand actually appears in the relevant M1 feature brief (tell Claude to patch the briefs where missing). Commit: `docs: milestone map + M2–M5 epic briefs`.

---


**Next:** read [05-feature-loop.md](05-feature-loop.md), then start F000 in [06-m1-plan.md](06-m1-plan.md). **Carry forward:** everything downstream obeys these artifacts — fix wrongness here, never later.
---
[◀ GitHub operations: environments, releases, lifecycle](03-github-operations.md) · [⌂ Index](README.md) · [The feature loop ▶](05-feature-loop.md)
