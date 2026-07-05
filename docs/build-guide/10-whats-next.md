> **MicLine Build Guide — Part 10 of 10** · [◀ Troubleshooting, cheat sheet, assumptions](09-troubleshooting.md) · [⌂ Index](README.md)

**In this part:** The look-up chapter — acknowledging what's built, then three horizons to draw from: product ideas, platform hardening, and process/repo evolution (including the open-to-contributors checklist). Read after the M1 gate; return after every milestone.
**Prerequisites:** v1.0.0 live (or any milestone freshly shipped). · **Your time:** ~10 min per visit; the ideas here execute through the normal machinery, never around it. *(your active attention; Claude runtime and limit pauses excluded)*

## §10 What's next — looking up after shipping

### A · Where you stand (let the system say it)

Don't take this file's word for what's done — the workflow was built so the repo states it: the **Releases page** is exactly what users have; **PROGRESS.md** + the milestone map are what you've completed and what's parked; the **Deployments sidebar** shows what DEV holds beyond the last release. The honest way to open any "what's next" session is the [file 09](09-troubleshooting.md) re-entry prompt — it produces the factual inventory this chapter builds on. What that inventory will show after M1 is worth pausing on: an empty repository became a versioned, publicly documented, zero-secret, two-environment SaaS with a live realtime core, shipped by one person steering an AI through gates. Everything below is *addition*, not correction — the machine you built is the asset; the ideas are just fuel for it.

**The one rule of this chapter:** nothing here bypasses the process. A seed you like becomes an issue → a brief → a kickoff-interview topic ([file 07](07-m2-to-m5.md)) or a [file 05](05-feature-loop.md) loop; a tooling idea takes the §7.12 lane. This file is a menu, not a backlog — the interviews decide.

### B · Product horizons (seeds for the M2+ kickoff interviews)

Each one line, deliberately underspecified — the P1-style challenge in the kickoff is where they earn their place:

- **Rented translation pipeline (TMS)** — already the named M2 candidate ([file 07 §9.1](07-m2-to-m5.md)); decides the fate of the optional Lunaria board.
- **More locales** — the F000 plumbing makes each new language a translation task, not a refactor (Constitution Art. IX); demand data from Discussions should pick them.
- **Event templates & recurring series** — a weekly meetup shouldn't re-enter settings; pairs naturally with the entitlement seam (per-series grants).
- **Co-moderators** — a second pair of hands on the pending queue; touches F002 roles and the F003 DO's role checks — interview hard before committing.
- **Post-event recap & export** — the `event_stats` snapshot rendered as a shareable recap page + CSV export; a natural Email Service use (send the recap to the moderator).
- **Embeds** — an iframe/read-only board view and an OBS-friendly overlay for livestream Q&A; new surface = new abuse review (F006 revisit is part of the brief).
- **Question merging / duplicate hints** — moderator-side "these three look alike"; keep it assistive, never automatic (Art. IV: server state stays authoritative and simple).
- **Audience polls** — the classic adjacent feature and a classic scope trap: it's *almost* a second product. If it survives the interview, it must reuse the DO/event model, not fork it.
- **Org workspaces / SSO** — only if paying-team demand appears; the Article-VI seam means it's grants, not new gating code.
- **Public API / webhooks** — turns MicLine into a building block; requires its own threat-model pass against [file 03 §G](03-github-operations.md) before any spec.
- **Accessibility as a feature** — a dedicated audit loop (screen reader, contrast, motion) beyond the per-surface checklists; cheap to run after M4, high trust value.

### C · Platform & infra hardening (mostly M5-adjacent; each is a chore-lane item or a small loop)

- **Per-PR preview deployments** — Workers supports uploading versions with preview URLs; a `deploy-preview.yml` would let you smoke a branch *before* merge instead of only on DEV. Verify current wrangler/versions support at use time.
- **Gradual (canary) rollouts** — already filed under M5 by [file 03 §D](03-github-operations.md); becomes worth it only when a bad deploy would hurt more than the reconnect flicker.
- **Scheduled D1 export** — a cron workflow running `wrangler d1 export` per environment into an artifact or R2 bucket; cheap insurance the rebuild runbook can point at ("data restore" is the one thing rebuild-from-zero can't conjure).
- **Synthetic uptime check + status page** — a scheduled Action hitting `/` and one API route on PRD, failing loudly; a public status page only if users ask.
- **Observability decision** — the M5 measurement loop ([file 07 §9.4](07-m2-to-m5.md)) owns this; the seed here is just: decide Workers Logs vs. Logpush *from measured need*, not from feature lists.
- **Rate-limiting binding re-check** — if F002 fell back to the KV-counter limiter, re-verify the native binding's availability and swap behind the same interface.
- **Playwright ring** — tech-context marks e2e "later"; "later" is when a regression escapes the three rings of [file 05](05-feature-loop.md), not before.
- **Load-test refresh** — re-run the M5 scripted-clients test at real growth thresholds (e.g., first 500-participant event booked), and feed results into the cost model.
- **Terraform/Pulumi re-evaluation** — trigger and criteria live in [file 02 §2.8](02-setup.md) / [file 07 §9.4](07-m2-to-m5.md): count the runbook's console-only lines at M5; decide with data.
- **Dependency policy review** — yearly: is grouped-weekly Dependabot still right; are all Action SHA-pins still maintained upstream.

### D · Process & repo evolution

**The open-to-contributors checklist** — run it *only* when you actively decide to open the repo; until then the [file 02 §2.4](02-setup.md) guardrails already keep drive-by noise near zero:

1. **Settle the CLA question first** ([file 02 §2.3](02-setup.md)): a CLA bot (e.g., a cla-assistant-style Action) or a stated "no outside PRs merged" policy — merging even one un-CLA'd PR freezes your relicensing freedom permanently.
2. **Adopt the EmDash agent-context pattern** ([file 08 §B](08-claude-code-reference.md)): move CLAUDE.md's content into `AGENTS.md`, then `ln -s AGENTS.md CLAUDE.md` — contributors' agents (which mostly read AGENTS.md) and your Claude Code now share one source of truth; nested CLAUDE.md files stay as they are.
3. **Add `CODE_OF_CONDUCT.md`** (Contributor Covenant is the boring, right default) — GitHub surfaces it automatically.
4. **Curate entry points:** label 3–5 genuinely small issues `good first issue` / `help wanted`; nothing invites unstructured PRs like an empty welcome mat.
5. **Lift the interaction limits** ([file 02 §2.4](02-setup.md) step 5) and open Discussions categories (Announcements, Q&A, Ideas) with a pinned "how we work" post linking CONTRIBUTING + this guide.
6. **Re-read CONTRIBUTING with stranger's eyes** — the issue-first rule, one-concern rule, translation-PR rule, and AI-disclosure rule were written for this day; tighten anything that assumed "contributors = future me".
7. **Re-verify the Actions posture:** fork-PR approval still strictest; consider restricting allowed Actions to a verified/pinned allow-list now that strangers can propose workflow changes.
8. **Decide issue hygiene for agents:** whether `taskstoissues`-generated issues are assignable by outsiders, and whether a triage label ("needs-maintainer-ack") gates work starting.

**Standing rituals** worth adopting regardless of contributors:

- **Per-milestone retro** — one fresh-session prompt after each release gate: `Read the milestone's specs, PRs, and decision-log entries. List: what the process caught, what escaped to DEV/PRD, which gate would have caught it cheaper, one concrete guide/CLAUDE.md/constitution amendment. Change nothing.` Accepted amendments go through the §7.12 chore lane.
- **Guide-freshness sweep** — at each milestone start, re-verify this guide's point-in-time facts (the README disclaimer lists them), run the [file 02 §2.9](02-setup.md) toolchain refresh, and update the [file 09](09-troubleshooting.md) assumptions log; the guide is a committed artifact, so it ages under the same rules as code.
- **Prune the inputs** — if `docs/inputs/` grows (legacy specs, research dumps), keep them, but make sure nothing but `docs/inputs/tech-decisions.md`'s provenance note claims any authority.

### E · How to choose (when everything above glitters)

A simple standing triage, applied at each "what now" moment: **(1) user pain first** — open bugs and Discussions feedback ride the bugfix lane ahead of everything; **(2) operability debt second** — anything the M5 audit ranked high protects your ability to keep going at all; **(3) differentiation third** — B-list seeds enter the next kickoff interview and fight for rank there; **(4) polish last** — C-list hardening items fill the gaps between loops and limit-pause weeks. Timebox pure exploration (a spike is a session, not a branch), and let the Releases page — not this file — be the record of what "done" means. The loop is the product now; MicLine is what it keeps shipping.

---


**Next:** nothing — this is the horizon file. **Carry forward:** seeds enter through interviews and lanes, never around them; re-read after every milestone.
---
[◀ Troubleshooting, cheat sheet, assumptions](09-troubleshooting.md) · [⌂ Index](README.md)
