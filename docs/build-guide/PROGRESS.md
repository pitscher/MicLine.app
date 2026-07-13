# MicLine.app — Build progress

**This is the single source of truth for where you are.** The guide files never track state; only this file does. It answers "where did I stop?" in one glance, so no session ends in a maze of half-remembered ToDos.

**The ritual (README, "Tracking your progress"):** at the end of every working session — (1) tick what you finished below, (2) set *Current position* to the next step, (3) commit: `git add docs/build-guide/PROGRESS.md && git commit -m "docs: progress"`. Returning after a longer pause: read this file, then run the re-entry protocol ([file 09](09-troubleshooting.md)).

> **Current position:** Phases 1–4 — product definition final; FR final pre-tech-interview review done 2026-07-13 (findings + operator decisions in decision-log §FR, corrections applied to all three artifacts); next: operator skim of the FR diff → Gate-1 commit `docs: product definition v1 finalized` (use `git add -A` — the docs/inputs/WIP file move is untracked), then P2 tech interview ([file 04 §4](04-product-definition.md))

> **Last updated:** 2026-07-13

> **Paused mid-feature?** If a `/speckit.implement` run was interrupted, the HANDOFF note lives in that feature's `specs/⟨NNN⟩/tasks.md` ([file 08 §D](08-claude-code-reference.md)) — this file only points there.

> **Work-package note:** the F-numbers below (F000–F007 legacy, F008–F013 provisional) are the [file 06](06-m1-to-m4-plan.md) engineering cut of the feature inventory's M1–M4 scope. Gate 2 (tech interview) finalizes that cut — if it re-cuts or renumbers, update the rows here to match.

## Phase 0 — Setup ([file 02](02-setup.md))

- [x] 01 read; `/model` output noted (which models your plan shows)
- [x] 02 §2.1 local tooling installed + sanity checks pass (node v24.x, pnpm 11.x, fnm hook in `~/.zshrc`)
- [x] 02 §2.2 Claude Code on Pro login; permission prompts ON
- [x] 02 §2.3–2.4 bootstrap commit pushed (LICENSE, skeleton, guide in-repo, README stub, CONTRIBUTING, manual-config-log started)
- [x] 02 §2.4 GitHub-side switches done **and logged** (secret scanning, push protection, priv. vuln. reporting, Issues+Discussions, fork-PR approval, 2FA both accounts, optional interaction limits)
- [x] 02 §2.4 throwaway pre-commit gitleaks hook in place
- [x] 02 §2.5 Spec Kit installed (pinned release + git extension); `/speckit` shows all 10 commands; committed
- [x] 02 §2.6 CLAUDE.md committed
- [x] 02 §2.9 read — the keep-current ritual (recurs at every milestone kickoff + after long pauses)
- [x] 03 read end-to-end once (§F is the lifecycle you'll live in)

## Phases 1–4 — Product definition ([file 04](04-product-definition.md))

- [x] P1 brainstorm complete → `docs: product definition v1` committed
- [x] P1.1 follow-up product interview complete (the product docs' provenance calls it **P2**) → brief/inventory/decision-log refined + committed
- [x] Milestone lineup revamped to **M1–M13 + SemVer targets** (product docs' provenance: **P3**) → product artifacts updated + committed
- [x] Build guide reconciled to the M1–M13 lineup (2026-07-12)
- [x] FR final pre-tech-interview review (2026-07-13) — skeptical senior-product pass; risk acceptances, capacity model, 14-day create-ahead default + truthfulness corrections applied (decision-log §FR)
- [x] Build-guide critical review (2026-07-13) — same pass over docs/build-guide; FR-sync (files 04/06/07), stale-window note (01/09), rollback row + tag-deploy rule (09/06), M4-gate FR-6 step (06 + this file)
- [ ] **Gate 1 (final):** read all three product artifacts end-to-end post-revamp → commit `docs: product definition v1 finalized`
- [ ] P2 tech interview → **Gate 2** passed → committed *(tech-decisions.md is provenance-only from here; work-package cut F000–F013 finalized here)*
- [ ] P3 constitution → **Gate 3** passed → committed
- [ ] P4 epic briefs M5–M13 + forward-compat demands → **Gate 4** passed → committed
- [ ] 03 §A milestone/label block run (13 GitHub Milestones + labels) + Roadmap Project board created via web UI (03 §A)

## M1 — Deployable foundation · `v0.1.0` ([file 06 §8.1](06-m1-to-m4-plan.md); each package = full [file 05](05-feature-loop.md) loop, merged, DEV smoked)

- [ ] **F000** foundations & pipeline — loop done · exit checklist: [ ] a `pnpm dev` hello · [ ] b CI green · [ ] c fake secret blocked · [ ] d **02 §2.7 row 1** done + deploy-dev re-run + dev.micline.app live · [ ] e branch protection ON
- [ ] **F008** platform config, flags & hardening baseline (FND-03/04/06)
- [ ] **Release `v0.1.0`** published + `production` approved + PRD responds (unannounced)

## M2 — Moderator identity and event setup · `v0.2.0` ([file 06 §8.2](06-m1-to-m4-plan.md))

- [ ] 02 §2.7 row 2 done + logged (email domain) + `SESSION_SECRET` from row 4 — *before F002*
- [ ] **F001** event creation & scheduling (EVT-01…04, MOD-03)
- [ ] **F002** moderator auth (AUTH-01/02) — incl. manual verify: real magic-link mail, SPF/DKIM/DMARC pass
- [ ] **Release `v0.2.0`** published + approved + PRD smoked

## M3 — Open-line event runtime · `v0.3.0` ([file 06 §8.3](06-m1-to-m4-plan.md))

- [ ] **F003** EventRoom realtime + open line — Phase A · Phase B (two implement runs)
- [ ] **F004** moderator console (checklist step run)
- [ ] **F005** participant surface (checklist step run)
- [ ] **F009** projector board, minimal (checklist step run)
- [ ] SAFE-03/04 content rules & profanity in place (inside F003/F005 or micro-loop — per Gate 2 cut)
- [ ] **M3 gate:** two-device internal-alpha smoke on DEV (full script in file 06 §8.3) · foundation revisit done (FND-01)
- [ ] **Release `v0.3.0`** published + approved + PRD smoked → 🎉 **internal end-to-end alpha**

## M4 — Public-beta readiness · `v0.4.0` ([file 06 §8.4](06-m1-to-m4-plan.md))

- [ ] 02 §2.7 row 3 (Turnstile widget) + `TURNSTILE_SECRET_KEY` from row 4 — *before F006*
- [ ] **F006** bot defense & rate limiting (SAFE-01/02)
- [ ] **F010** retention & purge machinery (EVT-06/07, OPS-03)
- [ ] **F011** recap & account self-service (EVT-05, AUTH-03/04)
- [ ] **F012** failure states, feedback & locale completion (PRT-08, MOD-04, I18N-02)
- [ ] **F007** marketing site & legal pages (MKT-01/02)
- [ ] **F013** Lunaria translation-status loop (FND-05 — non-gating; may run earlier)
- [ ] **M4 release gate** (file 06 §8.4, in order): [ ] 1 secret hygiene sweep · [ ] 2 every 02 §2.7 row confirmed · [ ] 3 full two-device DEV smoke incl. real email, Turnstile, deletion, countdown, both locales · [ ] 4 FR-6 additions (security-review pass · success-signals-or-no-targets decision · legal acknowledgment) · [ ] 5 `v0.4.0` release published + `production` approved · [ ] 6 PRD smoke · [ ] 7 go public (Discussions post, badges, roadmap pinned)
- [ ] 🎉 **`v0.4.0` — first responsible public beta live on micline.app**
- [ ] 10-whats-next.md first read (post-beta look-up; re-visit after every milestone)

## M5–M12 — post-beta milestones ([file 07](07-m5-to-m13.md); each: kickoff ritual → briefs gated → loops → release)

- [ ] **M5** Moderator control and data portability → `v0.5.0` (kickoff owns: export-format list, snapshot naming)
- [ ] **M6** Operator control plane → `v0.6.0` (admin-wall mechanism per tech interview OQ1)
- [ ] **M7** Emergency operations and recovery → `v0.7.0` (emergency copy set at spec)
- [ ] **M8** Entitlements, invite codes and lockdown → `v0.8.0` (kickoff owns OQ3 voucher UX) → *operationally controlled public service*
- [ ] **M9** Curated moderation → `v0.9.0` (kickoff owns filter matrix, curated defaults, co-mod mechanism) → *differentiated product*
- [ ] **M10** Extended event toolkit → `v0.10.0` (kickoff owns markdown-lite subset, TMS vs Lunaria, de-formal)
- [ ] **M11** Design system, identity and experience overhaul → `v0.11.0` (kickoff owns logo/colors OQ9, sound-cue decision)
- [ ] **M12** Operational maturity and compliance → **`v1.0.0`** (evidence gate: load test before degraded modes; rebuild-from-zero runbook; cost model; compliance pack) → 🎉 **first production-mature release**

## M13 — Monetization, if ever (provisional `v1.1.0`)

- [ ] Stays parked until you deliberately choose otherwise ([file 07 §9.9](07-m5-to-m13.md))
