> **MicLine Build Guide — Part 8 of 10** · [◀ M2–M5 playbooks](07-m2-to-m5.md) · [⌂ Index](README.md) · [Troubleshooting, cheat sheet, assumptions ▶](09-troubleshooting.md)

**In this part:** The reference: technique table, context & cost discipline, the AGENTS.md and plugin verdicts, and the Pro-limit workflow including the € escape-hatch math.
**Prerequisites:** None — skim after file 02, reread whenever sessions feel expensive. · **Your time:** ~15–20 min skim; a lifetime as a reference. *(your active attention; Claude runtime and limit pauses excluded)*

## A · Technique reference (when & why)

| Technique | Use it for | How |
|---|---|---|
| **/model** | [file 01](01-model-strategy.md) routing | Switch anytime, even mid-session |
| **/effort** | Reasoning depth ("ultrathink") | `low/medium/high/xhigh` persist across sessions; `max` = deepest, current session only (the menu's `ultracode` = xhigh plus auto-orchestrated workflows, also session-only — not used in this guide's routing). Marked per prompt in this guide |
| **Plan Mode** | Exploration you want guaranteed read-only: "how does X currently work", pre-refactor analysis in M5 | Shift+Tab to toggle. Not needed inside /speckit.\* steps (they're already artifact-first) |
| **/clear** | Between every step marked `[SESSION: fresh]` | Cheaper + better than compacting; all state is in files |
| **/context** | Long sessions (P1 interview, F003 implement) | If context is bloated → write state to files → /clear → resume |
| **Subagents (Task/Explore)** | "Research current Cloudflare docs for X while keeping my context clean" — plans for F002/F003 benefit | Ask: `Use a subagent to read ⟨llms.txt pages⟩ and report back only the API signatures we need` |
| **Paste images** | M4 visual iteration; debugging UI states | Drag/paste screenshots into the prompt |
| **Claude Design** | M4 visual work only — design system + surface explorations ([file 07 §9.3](07-m2-to-m5.md)) | claude.ai → Design; `/design` + `/design-sync` inside Claude Code. Shares your Pro usage limits — budget it; re-verify capabilities at M4 (research preview) |
| **`claude --continue` / `--resume`** | Reopen the most recent / a chosen past session after a limit pause | From the repo dir |
| **Hooks** | Already used: pre-commit gitleaks (git hook, simpler than a Claude hook and covers non-Claude commits too) | Add Claude-side hooks only if you later find a repeated failure mode |
| **MCP servers / plugins** | Not needed for this workflow — the CLAUDE.md llms.txt fetch protocol covers Cloudflare docs. Revisit in M5 if you want live account introspection (Cloudflare publishes official MCP servers) | `claude mcp add …` when the need is real, not before |
| **CLAUDE.md** | Standing rules ([file 02 §2.6](02-setup.md)) | Keep it short; move anything long into docs/ and link it. `/speckit.plan` appends stack context — leave its section intact |
| **/speckit templates** | If a template keeps producing something you dislike | Override per-project in `.specify/templates/overrides/` instead of fighting it in prompts |
| **Git worktrees / parallel sessions** | **Don't** (Pro limits + solo review). Revisit only if you move to Max | — |
| **Ask Claude Code about itself** | Any "can Claude Code do X?" | Just ask in-session; it fetches its own current docs |


## B · Context & cost discipline (your Pro plan's best friend)

The context window is the cost. Everything below shrinks what Claude re-reads every turn:

1. **CLAUDE.md stays small.** Target ≤ 150 lines. Every line is an instruction competing for the model's limited instruction-following budget — Claude Code's own system prompt already spends a chunk of it. Link, don't inline: `docs/engineering/tech-context.md` is referenced, never pasted. Never put linter-enforceable style rules in it — that's Biome's job, not the model's.
2. **Nested CLAUDE.md files** (created in F000): Claude Code loads a subdirectory's CLAUDE.md only when working there. Put DO-invariant reminders in `apps/web/worker/CLAUDE.md` and contract rules in `packages/shared/CLAUDE.md` — zero cost to sessions that never touch those trees.
3. **`@path/to/file` imports** inside CLAUDE.md pull a file into every session on load — use for at most one or two tiny always-relevant files; it's the easiest way to bloat every session.
4. **Files are the memory; the chat is not.** All state lives in `specs/`, `docs/`, `tasks.md`. That's why `[SESSION: fresh]` + `/clear` per step is cheaper AND better than long conversations or heavy `/compact` use (compaction is lossy right where you need precision).
5. **Point, don't paste.** Say `read worker/services/events.ts` instead of pasting it; Claude reads selectively. Never paste whole logs — save to a file and reference the path.
6. **Right-size `/effort`.** The [file 01](01-model-strategy.md) markings are ceilings: drop to `medium` for mechanical work; reserve `max` for the marked planning passes.
7. **Audit with `/context`** when a session feels sluggish or expensive — it shows exactly what occupies the window (including plugins; see §C).
8. **Auto memory:** Claude Code saves its own learnings (build quirks, commands) across sessions automatically — one more reason not to hand-duplicate such notes into CLAUDE.md.

**AGENTS.md — verified verdict (2026-07-04, still true 2026-07-05):** Claude Code does **not** natively read the cross-tool `AGENTS.md` standard; support remains an open feature request. You use exactly one agent, so: **CLAUDE.md only, no AGENTS.md.** Two triggers flip this later — adopting a second coding agent, or opening the repo to outside contributors (whose agents mostly *do* read AGENTS.md). When either fires, invert via the **EmDash pattern** (github.com/emdash-cms/emdash): AGENTS.md becomes the source of truth and `CLAUDE.md` becomes a symlink to it (`ln -s AGENTS.md CLAUDE.md`) — one file, every agent, zero sync burden. The move is item 2 of the open-to-contributors checklist ([file 10 §D](10-whats-next.md)).

## C · Plugins — verified recommendation (2026-07-04)

Claude Code has a plugin system, and the Anthropic-curated **official marketplace (`claude-plugins-official`) is available out of the box**: `/plugin` → *Discover* to browse, `/plugin install ⟨name⟩@claude-plugins-official` to install. Ground rules: every installed plugin adds standing context (= cost) to every session, and Anthropic curates but doesn't warrant third-party entries — so install nothing by default and prefer Anthropic-authored ones. For this project:

| Plugin | Verdict | Why |
|---|---|---|
| **typescript-lsp** (official marketplace, code-intelligence) | ✅ install at F000 | Feeds Claude live type errors/references from the real language server the moment it edits a file — in a strict-TS monorepo this catches mistakes before `tsc` runs, saving red commits and the tokens spent fixing them. Prereq binary: `npm i -g typescript-language-server typescript`. |
| **pr-review-toolkit** (Anthropic-authored, in the `anthropics/claude-code` marketplace) | ◻︎ optional at GATE D | Multi-agent PR review (tests, error handling, type design, simplification) with confidence scoring — a structured upgrade to the P5g hostile-review prompt. Cost caveat: multiple agents = multiple model runs, so use it on the risky PRs (F002/F003), not every merge. Add its marketplace once: `/plugin marketplace add anthropics/claude-code`. |
| GitHub integration plugin | ✖ skip | Redundant: the `gh` CLI already gives Claude everything (issues, PRs, releases) at zero standing-context cost. |
| Everything else (security linters, frontend/design skills, service integrations) | ✖ skip for now | Revisit design-oriented skills at the M4 kickoff and nothing earlier; add a plugin only after feeling the same pain twice. |

**Claude Code GitHub Action (`@claude` reviewing PRs in CI):** exists and is Anthropic-official, but it authenticates outside your interactive session — before ever enabling it, verify on the current Claude Code docs whether it can run on subscription auth or requires API pay-as-you-go billing, and what a review run costs. Given your "Pro subscription only" preference and that you personally review every PR at GATE D anyway, the default is **skip**.

## D · Working inside Pro limits

**What hitting a limit looks like:** Claude Code declines the next turn with a notice naming *which* limit (5-hour or weekly) and when it resets; nothing in flight is lost — file writes/commands either completed or didn't, and `git status` + `tasks.md` show exactly where reality stands. The 5-hour window is rolling (a coffee usually fixes it); the weekly cap resets on your plan's weekly boundary — `/usage` shows both meters.

- Budget rhythm: a 5-hour window comfortably fits one planning phase (file 04's P1–P4) **or** one implement run of a mid-size feature. F003 will span several windows — that's expected and fine.
- **Fable meter:** Fable 5 is capped at 50% of your weekly limit and drains fast. Check `/usage` (or the app's usage panel); when Fable is exhausted pre-July-7, drop to the post-window column of file 01 early.
- Safe-stop protocol when a limit hits mid-implement: tell Claude `Stop after the current task. Update tasks.md checkboxes and append a 5-line HANDOFF note (state, next task, open problems) to specs/⟨NNN⟩/tasks.md. Commit.` — resume later in a fresh session with `Read specs/⟨NNN⟩/{spec,plan,tasks}.md incl. HANDOFF, then /speckit.implement continue from the first unchecked task.` If it stopped uncleanly, `/speckit.converge` reconstructs reality.
- Weekly-limit weeks: shift to zero-Claude work — §2.7 manual steps, GATE reviews, manual smoke testing, reading `tech-context.md`. The workflow is designed so waiting never loses state.
- Escalation valve: usage credits (Settings → Usage) let you buy Fable/extra capacity per-token without leaving the subscription. Use deliberately (e.g., one Fable `max` pass on the F003 plan post-July-7), with a monthly cap set.

**€ math for the escape hatch (why prompts don't carry price tags).** This guide deliberately does *not* stamp a € estimate on every prompt: token consumption swings 5–10× with context size and session history, so per-prompt figures would be false precision that ages badly. Instead, price any prompt live in 30 seconds — Claude Code shows the session's token counts (`/usage` / statusline) — with: **cost = in-tokens × in-rate + out-tokens × out-rate**, per MTok: Sonnet 5 **$2 / $10** (intro to Aug 31, then $3/$15) · Opus 4.8 **$5 / $25** · Fable 5 **$10 / $50**; cache reads are ~90% off, and long sessions are *mostly* cache reads. Worked examples: a Fable `max` planning pass reading ~150k / writing ~20k ≈ 0.15×$10 + 0.02×$50 = **$2.50**; an Opus implement phase, ~400k in (largely cached) / 40k out ≈ **$2–4 effective**; a routine Sonnet task ≈ **$0.10–0.50**. Rule of thumb worth having before July-7 anxiety strikes: your *entire* M1 planning stack, run on metered Fable, would land around **$15–30** — annoying, not ruinous.


**Next:** bookmark [09-troubleshooting.md](09-troubleshooting.md). **Carry forward:** files are memory; `/effort` markings are ceilings; price prompts live, don't guess.
---
[◀ M2–M5 playbooks](07-m2-to-m5.md) · [⌂ Index](README.md) · [Troubleshooting, cheat sheet, assumptions ▶](09-troubleshooting.md)
