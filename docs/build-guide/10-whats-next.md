> **MicLine Build Guide — Part 10 of 10** · [◀ Troubleshooting, cheat sheet, assumptions](09-troubleshooting.md) · [⌂ Index](README.md)

**In this part:** The look-up chapter — acknowledging what's built, then three horizons to draw from: product ideas, platform hardening, and process/repo evolution (including the open-to-contributors checklist). First natural visit: after the M4 / `v0.4.0` public-beta gate; return after every milestone.
**Prerequisites:** any milestone freshly shipped. · **Your time:** ~10 min per visit; the ideas here execute through the normal machinery, never around it. *(your active attention; Claude runtime and limit pauses excluded)*

## §10 What's next — looking up after shipping

### A · Where you stand (let the system say it)

Don't take this file's word for what's done — the workflow was built so the repo states it: the **Releases page** is exactly what users have; **PROGRESS.md** + the feature inventory are what you've completed and what's parked; the **Deployments sidebar** shows what DEV holds beyond the last release. The honest way to open any "what's next" session is the [file 09](09-troubleshooting.md) re-entry prompt — it produces the factual inventory this chapter builds on. What that inventory will show after the M1–M4 run is worth pausing on: an empty repository became a versioned, publicly documented, zero-secret, two-environment SaaS with a live realtime core in public beta, shipped by one person steering an AI through gates. Everything below is *addition*, not correction — the machine you built is the asset; the ideas are just fuel for it.

**The one rule of this chapter:** nothing here bypasses the process. A seed you like becomes an issue → a brief → a kickoff-interview topic ([file 07](07-m5-to-m13.md)) or a [file 05](05-feature-loop.md) loop; a tooling idea takes the §7.12 lane. This file is a menu, not a backlog — the interviews decide.

### B · Product horizons (post-P3 state: most former seeds are decided)

The P2 final product interview and the P3 restructuring settled what an earlier draft of this section listed as open seeds. The current truth lives in two product-doc registers, not here:

- **Already scheduled — don't re-seed, just build the milestone:** recap & export (STD-05, M5) · co-moderators (STD-04, M9) · duplicate merge (STD-01, M9 — human-controlled, never automatic) · duplicate-event → templates (STD-07, M10) · rented TMS pipeline (STD-11, M10 — decides the fate of the M4 Lunaria board) · event branding (STD-10, M11) · self-hosting deployment paths (OPS-07, M12 — deploy button + C3 template + guide, decision-log AF-9).
- **The parking lot is the inventory's PRK-01…PRK-14 table** — each parked idea carries an explicit unparking condition (topics/tags, live reactions, waitlist, push notifications, request-to-speak, private notes, email change, …). A parked idea re-enters through its condition + a kickoff interview, never through nostalgia.
- **Closed by final product boundaries — needs a decision-log reopen, not a seed:** audience polls/quizzes/word clouds (not an engagement platform) · org workspaces/teams/seats/SSO (rejected — B2B stays "a bigger invite code") · embeds/OBS overlays/livestream ingestion (out of scope through `v1.0.0`) · recurring series/umbrella events (one event = one room/session) · waiting-time estimates (REJ-04) · fingerprinting of any kind (REJ-01). If one of these starts feeling tempting, that's a product-brief conversation, and this guide can't have it for you.

Seeds that are still genuinely open, deliberately underspecified — the P1-style challenge in a kickoff is where they earn their place:

- **More locales** — the F000 plumbing makes each new language a translation task, not a refactor (Constitution Art. IX); demand data from Discussions should pick them; the M10 TMS makes them cheap.
- **Accessibility as a feature** — a dedicated audit loop (screen reader, contrast, motion) beyond the per-surface checklists; cheap to run after M11, high trust value for the WCAG-AA principle.
- **Public API / webhooks** — turns MicLine into a building block; requires its own threat-model pass against [file 03 §G](03-github-operations.md) *and* a check against the integration-averse positioning before any spec.

### C · Platform & infra hardening (mostly M12-adjacent; each is a chore-lane item or a small loop)

- **Per-PR preview deployments** — Workers supports uploading versions with preview URLs; a `deploy-preview.yml` would let you smoke a branch *before* merge instead of only on DEV. Verify current wrangler/versions support at use time.
- **Gradual (canary) rollouts** — already filed under M12 by [file 03 §D](03-github-operations.md); becomes worth it only when a bad deploy would hurt more than the reconnect flicker.
- **Scheduled D1 export** — a cron workflow running `wrangler d1 export` per environment into an artifact or R2 bucket; cheap insurance the rebuild runbook can point at ("data restore" is the one thing rebuild-from-zero can't conjure — mind the 3-day content-purge promise when choosing what an export may contain).
- **Synthetic uptime check + status page** — a scheduled Action (or external monitor) hitting `/health` (FND-07 — exists from `v0.1.0`, auth-free, D1+KV-backed) and `/` on PRD, failing loudly; a public status page only if users ask (the FAQ's honest outage entry is the M4 baseline).
- **Observability decision** — the M12 measurement loop ([file 07 §9.8](07-m5-to-m13.md)) owns this; the seed here is just: decide Workers Logs vs. Logpush *from measured need*, not from feature lists.
- **Rate-limiting binding re-check** — if the M4 SAFE-02 work fell back to the KV-counter limiter, re-verify the native binding's availability and swap behind the same interface.
- **Playwright ring** — tech-context marks e2e "later"; "later" is when a regression escapes the three rings of [file 05](05-feature-loop.md), not before.
- **Load-test refresh** — re-run the M12 scripted-clients test at real growth thresholds (e.g., first 500-participant event booked), and feed results into the cost model + the OQ6 limits question.
- **Terraform/Pulumi re-evaluation** — trigger and criteria live in [file 02 §2.8](02-setup.md) / [file 07 §9.8](07-m5-to-m13.md): count the runbook's console-only lines at M12; decide with data.
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

A simple standing triage, applied at each "what now" moment: **(1) user pain first** — open bugs and Discussions feedback ride the bugfix lane ahead of everything; **(2) operability debt second** — anything the M12 audit ranked high protects your ability to keep going at all; **(3) differentiation third** — B-list seeds enter the next kickoff interview and fight for rank there; **(4) polish last** — C-list hardening items fill the gaps between loops and limit-pause weeks. Timebox pure exploration (a spike is a session, not a branch), and let the Releases page — not this file — be the record of what "done" means. The loop is the product now; MicLine is what it keeps shipping.

---


**Next:** nothing — this is the horizon file. **Carry forward:** seeds enter through interviews and lanes, never around them; re-read after every milestone.
---
[◀ Troubleshooting, cheat sheet, assumptions](09-troubleshooting.md) · [⌂ Index](README.md)
