# MicLine.app — Build Guide

**How MicLine.app is being built, end to end, by one person + Claude Code + GitHub Spec Kit.**

> ⚠️ **Point-in-time document.** Written **2026-07-04**, reviewed and corrected **2026-07-05**, reconciled on **2026-07-12** to the approved M1–M13 milestone lineup from the P3 product restructuring (`docs/product/feature-inventory.md` is the milestone authority), for a specific setup (solo developer, macOS, Claude Pro plan). Product facts inside — the Fable 5 subscription window ending **July 7, 2026**, model lineups, Spec Kit v0.12.x flags/commands (0.12.4 at verification), the local-toolchain pins (Node 24 Active LTS, pnpm 11 — repo-pinned per [file 02 §2.1](02-setup.md)), Cloudflare Email Service beta behavior, Claude Code plugin availability, Claude Design's preview status, Lunaria's beta status — were verified on those dates and *will* age. The workflow itself is built to survive that: it re-verifies external facts at the moment they're used (every plan step fetches current docs). If you're reading this in the future, trust the process, re-check the facts.

## What this is

A complete, self-sufficient walkthrough from an empty repository to a live, versioned SaaS on Cloudflare — including every prompt to paste into Claude Code, which model to use when, where to stop and review, and how a feature travels from idea to production once the product is live. It assumes no prior Cloudflare, AI-workflow, or large-SaaS experience; where a concept is likely new, the guide explains it in place (and the glossary below catches the rest).

## The journey

```
Phase 0            Phase 1–4 (planning)              per work package           ship
┌─────────┐   ┌──────────────────────────────┐   ┌──────────────────┐   ┌────────────┐
│ 02 setup│ ▶ │ 04 interviews → tech context │ ▶ │ 05 loop on each  │ ▶ │ Release    │
│ 03 read │   │   → constitution → epics     │   │ 06 M1–M4 packages│   │ v0.1.0 …   │
└─────────┘   └──────────────────────────────┘   └──────────────────┘   │ v0.4.0=beta│
                     (01 decides which model does what, throughout)     └────────────┘
     then: 07 runs M5–M12 the same way (→ v1.0.0 at M12; M13 parked) · 08/09 are references you'll revisit · 10 is where you look up after each milestone
```

## Files (reading order = numeric order)

| # | File | What's inside | Read when |
|---|---|---|---|
| 1 | [01-model-strategy.md](01-model-strategy.md) | Which Claude model + effort for which work; the July-7 Fable window plan | **First — today** |
| 2 | [02-setup.md](02-setup.md) | macOS tooling, Claude Code, AGPL-3.0, repo bootstrap, Spec Kit, CLAUDE.md, Cloudflare timeline, keep-current ritual (§2.9) | Once, day 1; §2.9 recurs |
| 3 | [03-github-operations.md](03-github-operations.md) | DEV/PRD environments, quality gates, Releases-as-changelog, the idea→PRD lifecycle, moderation & security hardening | Once now; §F/§G forever |
| 4 | [04-product-definition.md](04-product-definition.md) | Prompts P1–P4: product interviews (done) → tech context → constitution → M5–M13 epic briefs | Days 1–3 (Fable window) |
| 5 | [05-feature-loop.md](05-feature-loop.md) | The 10-step loop (P5a–P5g), how Claude drives GitHub, the three testing rings, how to work the gates, the bugfix + config lanes | Before F000, then every feature |
| 6 | [06-m1-to-m4-plan.md](06-m1-to-m4-plan.md) | M1–M4 (`v0.1.0`–`v0.4.0`) work packages in order, each with scope + plan addendum + special notes; the Lunaria loop; per-milestone release gates up to the public beta | Throughout M1–M4 |
| 7 | [07-m5-to-m13.md](07-m5-to-m13.md) | Kickoff ritual + playbooks M5–M12 (`v0.5.0`→`v1.0.0`): moderator control, admin, emergency ops, entitlements, curated moderation, toolkit, design overhaul, operational maturity — plus parked M13 monetization | At each milestone start |
| 8 | [08-claude-code-reference.md](08-claude-code-reference.md) | Techniques, context & cost discipline, AGENTS.md verdict, plugins verdict, Pro-limit workflow | Skim early; revisit often |
| 9 | [09-troubleshooting.md](09-troubleshooting.md) | Symptom→fix table, one-page cheat sheet, re-entry-after-a-pause protocol, assumptions log | When stuck, and after any long break |
| 10 | [10-whats-next.md](10-whats-next.md) | Looking up after shipping: product horizons, infra hardening, process evolution, the open-to-contributors checklist | After the M4 public-beta gate; again after every milestone |
| — | [PROGRESS.md](PROGRESS.md) | **Your single progress tracker** — the only file you edit to record where you are | Every session end; first thing after any pause |
| — | [tech-decisions.md](../inputs/tech-decisions.md) | The distilled, pre-verified stack decisions (input to prompt P2; provenance-only after Gate 2) — lives at `docs/inputs/` | Fed to Claude, not you |

## Conventions used everywhere

**Prompt blocks** are pasted into Claude Code verbatim; edit only `⟨angle-bracket⟩` placeholders. Their header tells you three things to set first: `[MODEL: x]` → `/model`, pick x · `[EFFORT: high|max]` → `/effort` (this is the current, official mechanism behind what used to be called "ultrathink" — `max` is the deepest setting and applies to the current session only) · `[SESSION: fresh]` → new session or `/clear`, because carrying stale context forward degrades quality and burns your Pro limit.

**🛑 GATE n** marks the points where *you* read an artifact and approve or correct it before anything continues — this is how the process stays exactly on your vision: you steer specs and plans, not diffs. **Fnnn** are engineering work packages of the M1–M4 arc ([file 06 §8.0](06-m1-to-m4-plan.md) maps them to milestones and feature-inventory IDs); **Pn** are prompts; **§** references within a file are that file's own sections unless a file number is named.

**💡 Nice to know boxes.** Skippable background — context, history, why-it-works — always appears as a blockquote starting with `💡 **Nice to know —**`. Read it once or skip it; nothing you must *do* is ever inside one.

**Milestones: the M1–M13 lineup is decided, and the Feature Inventory owns it.** The original guide treated milestones as working titles to be challenged at Gate 4; that challenge happened on the product side instead (the P3 milestone restructuring, 2026-07-12 — see `docs/product/decision-log.md` §P3). The canonical set is now **M1–M13 with one target release each** (`v0.1.0` … `v1.0.0` at M12; M13 provisional `v1.1.0`), executed **in numeric order**, with `docs/product/feature-inventory.md` as the authority for milestone ownership and target versions — every M-reference in this guide maps onto that lineup. Two watch-outs: (a) pre-P3 documents (and this guide's own history) used the same M-numbers for *different* milestones — inside preserved prompts and decision-log history, milestone numbers mean whatever the annotation says; (b) the guide's prompt numbers **P1–P5** are *not* the product docs' provenance labels P1/P2/P3 (there, P2 = the P1.1 follow-up interview and P3 = the milestone restructuring — [file 04](04-product-definition.md) disambiguates).

Everything lives in files (`.specify/`, `specs/`, `docs/`), so hitting a Pro usage limit mid-anything is safe — stop, resume later ([file 08 §D](08-claude-code-reference.md)).

## Tracking your progress

**[PROGRESS.md](PROGRESS.md) is the single tracker** — a granular checklist of every step, gate, and feature in this guide, plus a one-line "Current position" field. The guide files themselves never record state; only PROGRESS.md does.

**Session-end ritual (make it muscle memory):** before you stop for the day, tick what you finished in PROGRESS.md, set *Current position* to the next step, and commit: `git add docs/build-guide/PROGRESS.md && git commit -m "docs: progress"`. Coming back — after an hour or after months — starts by opening PROGRESS.md, then (after longer breaks) the re-entry protocol in [file 09](09-troubleshooting.md).

## Glossary (the terms this guide won't re-explain)

**Spec Kit** — GitHub's spec-driven-development toolkit; adds `/speckit.*` commands to Claude Code.

**Constitution** — the project's non-negotiable principles; every plan is checked against it.

**Spec / Plan / Tasks** — per-feature artifacts in `specs/⟨NNN⟩-*/`: the WHAT, the HOW, the ordered to-do list.

**Clarify / Analyze / Converge** — Spec Kit's question-asking, consistency-checking, and drift-detecting commands.

**Worker** — Cloudflare's serverless runtime (≈ Azure Functions at the edge).

**Durable Object (DO)** — a single-threaded, stateful, addressable mini-server (≈ Azure Durable Entities); one per live event holds the question line.

**D1 / KV** — Cloudflare's serverless SQLite database / eventually-consistent key-value store.

**Wrangler** — Cloudflare's CLI + config (`wrangler.jsonc`); its *environments* give us DEV and PRD.

**GitHub Environment** — a deploy target with its own secrets and (for `production`) a required-reviewer pause button.

**CLAUDE.md** — the file Claude Code reads at session start; this project's standing rules.

**AGENTS.md** — the cross-tool equivalent other coding agents read; not used while solo ([file 08 §B](08-claude-code-reference.md)), adopted via symlink if the repo opens to contributors ([file 10 §D](10-whats-next.md)).

**Lunaria** — an open-source, git-history-based translation-status dashboard; a light tooling loop (FND-05, M4) in [file 06](06-m1-to-m4-plan.md).

**Effort** — Claude's reasoning-depth dial (`/effort`: `low`–`xhigh` persist across sessions, `max` is session-only). The old "ultrathink" prompt keyword still nudges a single turn, but this guide standardizes on `/effort`.

---
Start with **[01-model-strategy.md](01-model-strategy.md)** — it tells you what to do in the next three days.
