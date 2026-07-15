# MicLine.app — Documentation map

> **The one entry point for humans and agents** (decision-log AF-10, 2026-07-15). This repository carries extensive documentation before its first line of code; this map says **where to look and which document wins**. Standing rule: any change that adds, moves, or retires a document updates this map in the same PR.

## Authority rules (who wins when documents overlap)

| Question | The one authoritative document |
|---|---|
| WHAT ships, in which milestone, under which version? | [`product/feature-inventory.md`](product/feature-inventory.md) — the single milestone/version authority (deliberately no separate milestone map) |
| WHY was something decided, rejected, or parked? | [`product/decision-log.md`](product/decision-log.md) (sections A–H, P2, P3, FR, AF + the open-questions/scheduled-decisions registry at its end) |
| WHAT is the product — positioning, model, principles, lifecycles? | [`product/product-brief.md`](product/product-brief.md) (Revision 5; its data-lifecycle table is constitutionally binding) |
| HOW is it built — stack, architecture, data model, security design? | [`engineering/tech-context.md`](engineering/tech-context.md) — read before **any** `/speckit.plan`; amendment records in §23/§24 |
| Enduring principles feeding the constitution? | [`product/constitution-input.md`](product/constitution-input.md) → `.specify/memory/constitution.md` (once P3 runs) |
| Per-feature WHAT/WHY for the M1–M4 work packages? | [`product/feature-briefs/`](product/feature-briefs/) (F000–F013; each names milestone, version, inventory IDs) |
| Where are we right now? | [`build-guide/PROGRESS.md`](build-guide/PROGRESS.md) — the **only** file that tracks state |
| HOW does the operator work — process, prompts, gates, lanes? | [`build-guide/`](build-guide/README.md) (files 01–10; **05 §7.0** = the intake funnel for any new idea; 05 §7 = the feature loop) |
| Pre-history / raw inputs? | [`inputs/`](inputs/) — **provenance only, never authority** (tech-decisions.md is superseded by tech-context.md since Gate 2) |

## Directory guide

| Path | Contents |
|---|---|
| `docs/product/` | Product brief · feature inventory · decision log · constitution input · `feature-briefs/` (per-work-package briefs) · `epics/` (M5–M13, arrives with prompt P4) |
| `docs/engineering/` | `tech-context.md` (the HOW) · `runcost-estimate.md` (list-price cost picture; superseded by the M12 measured model) |
| `docs/build-guide/` | The end-to-end operator guide (01 model strategy · 02 setup · 03 GitHub/env design · 04 product definition & reviews · 05 intake funnel + feature loop · 06 M1–M4 plan · 07 M5–M13 playbooks · 08 Claude Code reference · 09 troubleshooting · 10 what's next) + `PROGRESS.md` |
| `docs/runbooks/` | `resource-bootstrap.md` (Cloudflare resource creation — F000 completes) · `security-incident-response.md` · `maintenance.md` (recurring checks) · `manual-config-log.md` (every by-hand click, one line each) · `config-operations.md` (arrives with F008) |
| `docs/inputs/` | Archived provenance (`tech-decisions.md`, `WIP-feature-ideas.reconciled.md`) |
| `specs/` | Spec Kit per-feature artifacts (spec/plan/tasks) — exists from F000 on |
| `.specify/` | Spec Kit machinery: constitution, templates, workflows |
| `CLAUDE.md` (root) | Agent context: non-negotiables, ground-truth pointers, Cloudflare docs protocol, commands |
| `CONTRIBUTING.md` (root) | Ground rules for humans and agents (spec-first, tested, one concern per PR, AI disclosure, CLA) |

## Fast answers

- **"May I build this?"** → No feature code without a spec under `specs/` (constitution; CLAUDE.md). New ideas enter through the intake funnel: [`build-guide/05-feature-loop.md`](build-guide/05-feature-loop.md) §7.0.
- **"Is this decided, parked, or rejected?"** → decision log; never resurrect a REJ-\* item or unpark a PRK-\* item without its recorded condition/reopen.
- **"Which Cloudflare facts can I trust?"** → None from memory: fetch current docs per the CLAUDE.md protocol; tech-context §22 lists what to re-verify when.
- **"What does a config value do / how do I change it?"** → `docs/runbooks/config-operations.md` (from F008); mechanism in tech-context §13.
- **"What did a review change?"** → tech-context §23 (2026-07-14) and §24 + decision-log §AF (2026-07-15).
