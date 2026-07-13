> **MicLine Build Guide — Part 1 of 10** · [⌂ Index](README.md) · [One-time setup (macOS) ▶](02-setup.md)

**In this part:** Which model does what, and the three-day plan for the Fable 5 window.
**Prerequisites:** None — this is the first read. · **Your time:** ~10 min to read; the plan it sets runs through file 04. *(your active attention; Claude runtime and limit pauses excluded)*

## §1 Model strategy — read this first (deadline: **July 7, 2026**)

**Verified facts (as of July 4, 2026):**

- **Fable 5** is included on your Pro plan for up to **50% of your weekly usage limit through July 7 only**. After July 7 it moves to pay-per-token usage credits (~$10/$50 per MTok — the "pricy pay-as-you-go" you mentioned). Anthropic has said it *intends* to restore Fable to subscriptions "as capacity allows," but there is no date. Treat July 7 as a hard cutoff.
- **Sonnet 5** (released June 30, 2026) is the new Pro default in Claude Code. It closes most of the gap to Opus 4.8 at far lower usage-limit drain, with selectable effort levels. This is your workhorse.
- **Opus 4.8** is the strongest non-Fable model. Availability in Claude Code's `/model` picker can vary by plan — check yours ([file 02](02-setup.md), step 2.2.4). If it's selectable on your Pro plan, use it per the routing table; if not, Sonnet 5 at `/effort high`/`xhigh` is the escalation path, and Opus via usage credits is the emergency option.
- Fable 5 drains your limit noticeably faster than other models — don't route routine work through it just because it's included.

> **2026-07-13 note (final-review pass):** July 7 has passed. Route by what `/model` actually offers ([file 02 §2.2](02-setup.md) step 4): if Fable 5 appears on your plan, the Fable rows below still apply; if not, use the right-hand column and the substitution rule under the routing table. The remaining P2–P4 prompt headers name Fable as the *ideal* — substitute per this table when it isn't available; [file 08 §D](08-claude-code-reference.md) prices the metered fallback.

**Consequence — the Fable window plan (July 4–7):** planning artifacts are durable; code is regenerable. Spend the window on artifacts, not code.

| Do **before July 7** with Fable 5 | Do **after July 7** with Opus 4.8 / Sonnet 5 |
|---|---|
| Product brainstorm (P1 — [file 04](04-product-definition.md)) | All `/speckit.implement` runs |
| Establish tech context (P2 — file 04) | Remaining specify/clarify/plan for later features |
| Constitution (P3 — file 04) | Everything in M2–M13 |
| Epic briefs M5–M13 (P4 — file 04) | |
| `/speckit.specify` + `clarify` + `plan` + `analyze` for as many early work packages as the window allows, in the [file 06](06-m1-to-m4-plan.md) order | |

**Standing model routing table** (used throughout; "Opus 4.8" = substitute "Sonnet 5 @ effort xhigh" if Opus isn't in your picker):

| Work type | Model | Effort | Why |
|---|---|---|---|
| Brainstorm, constitution, epic briefs | Fable 5 (→ Opus 4.8 after Jul 7) | max | Highest-leverage durable artifacts; reasoning quality compounds through everything downstream |
| /speckit.specify, clarify, plan, analyze, converge | Fable 5 (→ Opus 4.8) | high–max | Spec/plan errors are the most expensive kind |
| /speckit.implement — hard features (F003 EventRoom, F002 auth) | Opus 4.8 | high | You said you'll trade speed for quality; these features carry the correctness risk |
| /speckit.implement — routine features (F005–F007, config, tests, docs) | Sonnet 5 | medium–high | Near-Opus quality, far less limit drain |
| Quick fixes, formatting, commit messages, small edits | Sonnet 5 | low–medium | Don't burn premium tokens on mechanical work |

Switch models mid-session with `/model` — e.g., start a debugging session on Sonnet 5, hit a genuinely hard bug, `/model` → Opus 4.8 for that one task, switch back.

---


**Next:** [02-setup.md](02-setup.md) — get the machine and repo ready. **Carry forward:** the routing table (you'll set `/model` + `/effort` per prompt from now on) and the July 7 cutoff.
---
[⌂ Index](README.md) · [One-time setup (macOS) ▶](02-setup.md)
