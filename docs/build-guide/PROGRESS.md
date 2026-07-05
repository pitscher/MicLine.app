# MicLine.app — Build progress

**This is the single source of truth for where you are.** The guide files never track state; only this file does. It answers "where did I stop?" in one glance, so no session ends in a maze of half-remembered ToDos.

**The ritual (README, "Tracking your progress"):** at the end of every working session — (1) tick what you finished below, (2) set *Current position* to the next step, (3) commit: `git add docs/build-guide/PROGRESS.md && git commit -m "docs: progress"`. Returning after a longer pause: read this file, then run the re-entry protocol ([file 09](09-troubleshooting.md)).

> **Current position:** ⟨not started — begin with 01-model-strategy.md⟩
> **Last updated:** ⟨YYYY-MM-DD⟩
> **Paused mid-feature?** If a `/speckit.implement` run was interrupted, the HANDOFF note lives in that feature's `specs/⟨NNN⟩/tasks.md` ([file 08 §D](08-claude-code-reference.md)) — this file only points there.

## Phase 0 — Setup ([file 02](02-setup.md))

- [ ] 01 read; `/model` output noted (which models your plan shows)
- [ ] 02 §2.1 local tooling installed + sanity checks pass (node v24.x, pnpm 11.x, fnm hook in `~/.zshrc`)
- [ ] 02 §2.2 Claude Code on Pro login; permission prompts ON
- [ ] 02 §2.3–2.4 bootstrap commit pushed (LICENSE, skeleton, guide in-repo, README stub, CONTRIBUTING, manual-config-log started)
- [ ] 02 §2.4 GitHub-side switches done **and logged** (secret scanning, push protection, priv. vuln. reporting, Issues+Discussions, fork-PR approval, 2FA both accounts, optional interaction limits)
- [ ] 02 §2.4 throwaway pre-commit gitleaks hook in place
- [ ] 02 §2.5 Spec Kit installed (pinned release + git extension); `/speckit` shows all 10 commands; committed
- [ ] 02 §2.6 CLAUDE.md committed
- [ ] 02 §2.9 read — the keep-current ritual (recurs at every milestone kickoff + after long pauses)
- [ ] 03 read end-to-end once (§F is the lifecycle you'll live in)

## Phases 1–4 — Product definition ([file 04](04-product-definition.md))

- [ ] P1 brainstorm complete → **Gate 1** passed → `docs: product definition v1` committed
- [ ] P2 tech context → **Gate 2** passed → committed *(tech-decisions.md is provenance-only from here)*
- [ ] P3 constitution → **Gate 3** passed → committed
- [ ] P4 milestone cut challenged + map & epics → **Gate 4** passed → committed
- [ ] 03 §A milestone/label block run (Gate-4 titles)

## M1 — feature loops ([file 06](06-m1-plan.md); each = full [file 05](05-feature-loop.md) loop, merged, DEV smoked)

- [ ] **F000** foundations & pipeline — loop done · exit checklist: [ ] a `pnpm dev` hello · [ ] b CI green · [ ] c fake secret blocked · [ ] d **02 §2.7 row 1** done + deploy-dev re-run + dev.micline.app live · [ ] e branch protection ON
- [ ] **F001** event lifecycle
- [ ] 02 §2.7 rows 2–4 done + logged (email domain, Turnstile, runtime secrets) — *before F002*
- [ ] **F002** moderator auth — incl. manual verify: real magic-link mail, SPF/DKIM/DMARC pass
- [ ] **F003** EventRoom realtime — Phase A · Phase B (two implement runs)
- [ ] **F004** moderator console UI (checklist step run)
- [ ] **F005** participant experience (checklist step run)
- [ ] **F006** abuse & content safety
- [ ] **F007** marketing site
- [ ] *(optional)* Lunaria translation-status tooling loop ([file 06](06-m1-plan.md))

## M1 release gate ([file 06](06-m1-plan.md)) — in order

- [ ] 1 secret hygiene sweep · [ ] 2 every 02 §2.7 row confirmed · [ ] 3 two-device DEV smoke incl. real email · [ ] 4 `v1.0.0` release published + `production` approved · [ ] 5 PRD smoke · [ ] 6 go public (Discussions post, badges, roadmap pinned)
- [ ] 🎉 **v1.0.0 live on micline.app**

## After M1 ([file 07](07-m2-to-m5.md) · [file 10](10-whats-next.md))

- [ ] 10-whats-next.md first read (post-M1 look-up)
- [ ] M2 kickoff ritual → briefs gated → loops… → M2 release (`v2.0.0`)
- [ ] M4 kickoff (design brief → tooling check → design system → surface loops) → release
- [ ] M5 kickoff (observability → constitution audit → retention → load test → **rebuild-from-zero runbook** → cost model) → release
- [ ] M3 stays parked until you deliberately choose otherwise
- [ ] Re-visit file 10 after each milestone (seeds → kickoff interviews, never ad hoc)
