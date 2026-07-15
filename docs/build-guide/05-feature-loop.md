> **MicLine Build Guide — Part 5 of 10** · [◀ Product definition (interviews → tech context → constitution → epics)](04-product-definition.md) · [⌂ Index](README.md) · [M1–M4 execution plan ▶](06-m1-to-m4-plan.md)

**In this part:** The loop every feature runs: 10 steps, how Claude drives GitHub for you, the three testing rings that protect `main`(=DEV), how to work each gate (incl. the reusable PG critic prompt), the post-launch bugfix lane, and the lighter lanes for config-only changes (§7.12).
**Prerequisites:** Gates 1–4 passed · milestones + labels created ([03 §A](03-github-operations.md) block). · **Your time:** ~25 min read; thereafter each feature costs roughly 1–3 h of your attention spread across its loop. *(your active attention; Claude runtime and limit pauses excluded)*

## §7 The feature loop (repeat for every feature, F000 → …)

Ten steps (0–9). **Step 0** hooks the loop into the GitHub lifecycle of [file 03 §F](03-github-operations.md): every feature starts as a GitHub issue — create one from the feature-request template (or pick the existing one), assign it to the feature's Milestone, note its number for the PR.

Steps 1–7 are planning (cheap to redo, do them well); step 8 is code; step 9 closes the loop. One feature at a time — **no parallel features**: on a Pro plan, with you as the single reviewer, parallelism trades your scarcest resources (limits + attention) for speed you don't need.

**Git shape:** `/speckit.specify` creates branch `⟨NNN⟩-⟨slug⟩` + `specs/⟨NNN⟩-⟨slug⟩/`. Stay on that branch through step 8, open a PR, merge via the PR after step 9. (The branching comes from Spec Kit's **git extension** — opt-in since v0.10.0, which is why [file 02 §2.5](02-setup.md) installs it explicitly. If specify ever stops branching, `specify extension add git` restores it.)

**How the loop touches GitHub (the mechanics you asked about).** You never leave the terminal for routine GitHub work — Claude Code drives the `gh` CLI whenever you ask, and you approve each command it proposes. Step 0 in practice is one paste:

```text
Create the GitHub issue for ⟨F00x name⟩: use the feature_request template's
fields, body = the summary paragraph from docs/product/feature-briefs/⟨file⟩
plus a link to it, milestone ⟨M1⟩, label type:feature. Print the issue number.
```

Claude runs `gh issue create …`; the same pattern covers issue updates/comments, opening the PR (step 9), and `gh release create`. The Projects board needs no prompting at all — its built-in workflows ([file 03 §A](03-github-operations.md)) auto-add new issues and flip them to Done when the linked PR merges. Division of labor: **you** decide what exists, **Claude** does the clerical `gh` calls, **GitHub automation** keeps the board honest.

**Where code is exercised before users see it — three rings.** Ring 1, your machine: `pnpm dev` boots the *entire* stack locally in the Workers runtime (miniflare, via the Vite plugin) — Worker, Durable Objects, D1, KV all faithfully emulated, email simulated to the console — so you click through every feature on localhost first (and inspect or edit the local KV/D1/DO state in the browser via Cloudflare's **Local Explorer** at `/cdn-cgi/explorer`, on by default — tech-context §18); `pnpm test` runs the same runtime headlessly (the vitest workers pool — home of the F003 state-machine suite). Ring 2, the PR: the five CI checks re-run everything on a clean machine, and branch protection means `main` physically cannot receive a red commit — this is what makes main==DEV trustworthy. Ring 3, DEV: the merge auto-deploys dev.micline.app, the first contact with *real* Cloudflare infrastructure (real DO placement, real D1, real email); your smoke test there is the last gate before a change becomes release-eligible. So: `main` only breaks if a behavior had no test — which is precisely what Gate C and the coverage rule exist to prevent.

| # | Step | Command / action | Model ([file 01](01-model-strategy.md) table) | Gate |
|---|---|---|---|---|
| 1 | Specify | `/speckit.specify` + brief (P5a) | Fable→Opus | — |
| 2 | Clarify | `/speckit.clarify` (P5b) | Fable→Opus | 🛑 G-A after |
| 3 | Plan | `/speckit.plan` + tech pointer (P5c) | Fable→Opus | 🛑 G-B after |
| 4 | Checklist | `/speckit.checklist` (P5d — UI/complex features) | Sonnet 5 | — |
| 5 | Tasks | `/speckit.tasks` | Sonnet 5 | — |
| 6 | Analyze | `/speckit.analyze` (P5e) | Fable→Opus | 🛑 G-C after |
| 7 | Issues (optional) | `/speckit.taskstoissues` | Sonnet 5 | — |
| 8 | Implement | `/speckit.implement` (P5f) | per file-01 routing | approve permission prompts |
| 9 | Converge + review + merge | `/speckit.converge`, PR review (P5g) | Fable→Opus review, Sonnet fixes | 🛑 G-D before merge |

### 7.1 Specify

```text
PROMPT P5a — specify ⟨FEATURE⟩
[MODEL: Fable 5 / Opus 4.8] [EFFORT: high] [SESSION: fresh]

/speckit.specify Read docs/product/feature-briefs/⟨FNNN-name⟩.md and
docs/product/product-brief.md, then write the specification for this feature.
Scope exactly what the brief scopes — flag scope you think is missing rather
than adding it. Requirements must be testable. Do not mention the tech stack.
```

### 7.2 Clarify — *this is your "ask me everything" step*

```text
PROMPT P5b — clarify
[SAME SESSION as P5a]

/speckit.clarify Interview me exhaustively. Prioritize questions where a wrong
guess would be expensive to reverse. Where the answer already exists in
docs/product/decision-log.md, cite it instead of re-asking. Number questions;
I answer by number.
```

Answer honestly, including "I don't know — recommend and mark ASSUMED". Clarify records answers into the spec's Clarifications section.
**🛑 GATE A:** read `specs/⟨NNN⟩/spec.md` top to bottom. This is the single highest-leverage review in the whole workflow — a minute here saves an hour in step 8. Commit the spec.

### 7.3 Plan

```text
PROMPT P5c — plan
[SAME SESSION or fresh] [EFFORT: high–max for F002/F003]

/speckit.plan Implement this feature per docs/engineering/tech-context.md
(binding: stack table, architecture rules, secrets matrix)
and the constitution. Before writing the plan, fetch the current Cloudflare
docs for every Cloudflare product this feature touches
(developers.cloudflare.com/⟨product⟩/llms.txt) and reconcile them against
tech-context.md — record any discrepancy in the plan's research section and
update tech-context.md. ⟨FEATURE-SPECIFIC ADDENDUM from file 06⟩
```

**🛑 GATE B:** read `plan.md` (+ `research.md`, `data-model.md`, `contracts/` if generated). You're a DevOps engineer — you are *qualified* to judge this layer; push back in-session until the plan is one you'd defend. Commit.

### 7.4 Checklist (run for F003–F005, F007, and anything UI-heavy)

```text
PROMPT P5d — checklist
/speckit.checklist Generate a requirements-quality checklist for this feature
focusing on: ⟨for UI features: accessibility, mobile/dark-hall usability,
empty/error/reconnect states⟩ ⟨for F003: state-machine edge cases, concurrent
mutation, reconnection/snapshot consistency⟩. Then evaluate the spec against it
and fix the spec where it fails.
```

### 7.5 Tasks → `/speckit.tasks` (no extra prompt needed). Skim `tasks.md` for ordering sanity (tests-before/with-code, small commits).

### 7.6 Analyze

```text
PROMPT P5e — analyze
/speckit.analyze Report every inconsistency, coverage gap, constitution
violation, and ambiguity across spec/plan/tasks. Then propose concrete fixes
and apply the ones I approve.
```

**🛑 GATE C:** approve the fixes. This is the last cheap exit before code.

### 7.7 Issues (optional but recommended from F001 on): `/speckit.taskstoissues` — turns `tasks.md` into GitHub issues (needs `gh auth login`; assign them to the feature's Milestone). Gives you progress visibility outside the terminal — and the public roadmap board ([file 03 §E](03-github-operations.md)) fills itself.

### 7.8 Implement

```text
PROMPT P5f — implement
[MODEL: per file-01 routing — Opus 4.8 for F002/F003, Sonnet 5 otherwise]
[EFFORT: high for hard features, medium otherwise] [SESSION: fresh]

/speckit.implement Work through tasks.md strictly in order. Rules:
- Commit after each completed task (conventional commits), checking the task
  off in tasks.md in the same commit.
- Run `pnpm test && pnpm typecheck && pnpm lint` before every commit; never
  commit red.
- Constitution Article I: if you are about to write anything resembling a
  secret or real email address, stop and use a placeholder + .dev.vars.example.
- If a task turns out to be impossible or wrong as written, STOP and tell me —
  do not improvise around the plan.
- Stop at the end of each task PHASE and summarize before continuing.
```

Big features: scope runs — `/speckit.implement Execute Phase 1–2 of tasks.md only, then stop.` Review, `/clear`, continue with the next phases in a fresh session (tasks.md checkboxes carry the state). Keep permission prompts on and actually read the diffs you approve — that's your inline code review.

### 7.9 Converge, review, merge

```text
PROMPT P5g — converge + self-review
[MODEL: Fable 5 / Opus 4.8] [EFFORT: high] [SESSION: fresh]

/speckit.converge
Then, acting as a hostile senior reviewer, review the full branch diff
(`git diff main...HEAD`) against spec.md, plan.md, and the constitution:
correctness, the Article-IV invariants, security (authZ on every mutation,
input validation, secret hygiene), and maintainability for a solo operator.
Output a findings list ranked by severity. Fix approved findings.
```

If converge appended tasks → run `/speckit.implement` again → converge again, until it reports converged. Then `gh pr create --fill` and add `Closes #⟨feature issue⟩` to the body; read the PR diff yourself on GitHub (**🛑 GATE D** — your final course-correction point; optionally run the pr-review-toolkit plugin here, [file 08 §C](08-claude-code-reference.md)); merge. The merge auto-deploys **DEV** — smoke it on dev.micline.app. PRD only changes when you publish a Release ([file 03 §F](03-github-operations.md)). Delete branch, `git checkout main && git pull`, next feature.

### 7.10 Working the gates (A–D and 1–4) — read yourself, then weaponize a critic

A gate is a judgment point, not a chore: **you read the artifact first**, because the gates exist for the one thing only you hold — the vision. Then, optionally, enlist Claude as a critic — but in a **separate fresh session**, so the author isn't grading its own homework:

```text
PROMPT PG — gate critique (reusable at any gate)
[MODEL: strongest available] [EFFORT: high] [SESSION: fresh — never the session that wrote the artifact]

Act as an adversarial reviewer. Read ⟨artifact path⟩ against
docs/product/product-brief.md, the constitution, and ⟨its spec/plan, if any⟩.
Do NOT edit anything. Output: (1) issues ranked by severity, (2) questions a
careful reader would ask, (3) what's missing. Be specific. No praise.
```

You triage the findings, then hand the accepted ones to the working session ("Apply findings 1, 3 and 4: ⟨paste⟩"). Per gate type: **Gates 1–3 and A** (documents) — PG as-is. **Gate B** (plan) — PG *plus* your own DevOps read; this is the layer you're professionally qualified to judge. **Gate C** — the analyze output already *is* the critique; you only approve fixes. **Gate D** (PR) — read the diff on GitHub yourself; P5g (and optionally pr-review-toolkit) already played critic. **UI gates** (checklist steps, all of M11) — screenshot the screen (⌘⇧4), annotate it in Preview/Markup, paste the image straight into Claude Code with your change requests: annotated pixels beat paragraphs for design feedback.

> 💡 **Nice to know —** there's no Anthropic "document critique" product to install — a fresh session with PG *is* the mechanism for text artifacts. For *visual* work, however, **Claude Design** does exist (Anthropic Labs research preview since April 2026, included with Pro, and it syncs with Claude Code): it's deliberately held back until the M11 design milestone, where [file 07 §9.7](07-m5-to-m13.md) covers when and how to evaluate it. Until then, the screenshot loop above is faster and costs nothing extra.

### 7.11 The bugfix lane (post-launch)

Not everything deserves the full loop. A reported bug is behavior diverging from an already-specced state, so: issue (bug template) → fresh session, Sonnet 5 → `Reproduce issue #N with a failing test first, then make it pass; touch nothing else` → PR (`Closes #N`) → Gate D → merge → ships in the next PATCH release. This is constitution-compatible: Article VIII governs *new* behavior, and a bugfix restores specified behavior — the failing test is its spec. If triage reveals the "bug" is actually new scope, it graduates to a feature brief and the full loop.

### 7.12 The config & chore lanes — when the change isn't (only) app code

Your question "how does Spec Kit handle changes that live only on the Cloudflare side, or only on the GitHub side?" has a structural answer: **the full loop governs repo-tracked *behavior*; everything lighter falls into one of the three lanes below.** The deciding rule: *if a change can be expressed in the repo, it must be; if it can't, it must be logged.*

| Lane | For | How it runs | Governance |
|---|---|---|---|
| **Chore lane** (repo-tracked, no product behavior) | Workflow/CI tweaks, issue/PR templates, lint config, dependency bumps Dependabot missed, docs, `wrangler.jsonc` edits, repo tooling adjustments (the Lunaria loop itself runs as a light feature loop — [file 06 §8.4](06-m1-to-m4-plan.md)) | Optional issue → branch → change → PR labeled `type:chore`/`type:docs` → Gate D → merge. No spec/plan/tasks — Article VIII governs *feature behavior*, and these have none | Still through CI + PR, **never** straight to `main`; the merge auto-deploys DEV like everything else |
| **GitHub-side only** (no repo artifact possible) | A repo Setting, enabling a feature, Project-board config, interaction limits | Do it in the web UI (or via `gh`), then **one line in `docs/runbooks/manual-config-log.md`** | The log line *is* the artifact — it's what makes the M12 rebuild runbook true |
| **Cloudflare-side only** (no repo artifact possible) | Dashboard switches, secret rotation (`wrangler secret put`), Turnstile widget edits, plan changes | Same: hand + log line. But first ask: *can* this live in `wrangler.jsonc` or a documented CLI line instead? If yes, that's the chore lane — the IaC posture ([file 02 §2.8](02-setup.md)) prefers it | Log line, plus the runbook check at M12 |

**Mixed changes** — a feature whose plan requires console clicks (F000 and F002 are the models) — need no special lane: the spec/plan covers the repo parts, and the manual steps become explicit **exit-checklist items**, each ending in a log line, so `/speckit.implement` never silently "does" something only you can do. If a chore starts sprouting acceptance criteria or user-visible behavior mid-flight, stop: it just became a feature — brief + full loop.

---


**Next:** [06-m1-to-m4-plan.md](06-m1-to-m4-plan.md) — run this loop on F000. **Carry forward:** the PG pattern; fresh session per step; you steer specs and plans, not diffs.
---
[◀ Product definition (interviews → tech context → constitution → epics)](04-product-definition.md) · [⌂ Index](README.md) · [M1–M4 execution plan ▶](06-m1-to-m4-plan.md)
