> **MicLine Build Guide — Part 9 of 10** · [◀ Claude Code reference: context, cost, plugins](08-claude-code-reference.md) · [⌂ Index](README.md) · [What's next ▶](10-whats-next.md)

**In this part:** Symptom→fix table, the one-page cheat sheet, the re-entry-after-a-pause protocol, and the assumptions log worth challenging.
**Prerequisites:** None. · **Your time:** Reference only. *(your active attention; Claude runtime and limit pauses excluded)*

## Troubleshooting

| Symptom | Fix |
|---|---|
| `/speckit.*` commands missing | You're not in the repo dir, or init didn't run: `specify init --here --force --integration claude`, restart Claude Code |
| `specify: command not found` | `uv tool` bin dir not on PATH → `uv tool update-shell` or add `~/.local/bin` to PATH |
| Tutorials say `specify init --ai claude` and it errors | Flag removed in v0.10.0 → use `--integration claude` |
| specify didn't create a branch/dir | The git extension is missing (opt-in since v0.10.0; [02 §2.5](02-setup.md) installs it): `specify extension add git` — or check `.specify/scripts/bash/create-new-feature.sh` ran (needs a git repo) |
| `corepack: command not found` | Expected on a current machine: corepack shipped only with Node < 25, and Homebrew's `node` doesn't include it at all. This guide doesn't use corepack — pnpm comes from brew and runs the exact version pinned in `package.json` `packageManager` by itself ([02 §2.1](02-setup.md)) |
| `node -v` isn't the version [02 §2.1](02-setup.md) installed | fnm's shell hook is missing, or another Node shadows it (typically brew's `node` — a pnpm dependency — which tracks the newest "Current" line, 26.x today): add `eval "$(fnm env --use-on-cd)"` to `~/.zshrc`, open a new shell; `which -a node` shows the resolution order |
| Implement drifted from plan | Stop; `/speckit.converge`; review appended tasks; re-implement. Never "just fix it live" for anything non-trivial (Article VIII) |
| Opus 4.8 not in `/model` on your plan | Use Sonnet 5 `/effort xhigh` as the escalation tier; usage credits for the rare Fable/Opus pass |
| Email never arrives locally | Expected: local dev **simulates** sends — read the magic link from the wrangler dev log/local file, or set `remote: true` on the send_email binding |
| Email Service beta pricing/limits turn bad | Swap the provider inside `worker/services/email.ts` (the Article-II seam exists for exactly this); candidates: Resend/Postmark via REST |
| CI deploy fails on auth | Token scopes (§2.7) or secrets misnamed; never debug by printing secrets in CI |
| Claude Code cites stale Cloudflare APIs | Enforce the CLAUDE.md docs protocol: `Fetch developers.cloudflare.com/⟨product⟩/llms.txt before answering` |
| Hit limit mid-step | [file 08 §D](08-claude-code-reference.md) safe-stop; artifacts on disk mean nothing is lost |
| `deploy-dev.yml` red right after F000 | Expected until [file 02 §2.7](02-setup.md) row 1 is done (Cloudflare secrets in the GitHub `dev` environment) — then re-run the workflow |
| Published a Release but PRD didn't deploy | The workflow triggers on `release: published` (not draft/created); and check whether the Actions run is *waiting* for the `production` environment approval |
| Can't find where to approve a PRD deploy | Repo → Actions → the deploy-prd run → yellow "Review pending deployments" banner → tick `production` → Approve and deploy |
| Plugin "not found in any marketplace" | `/plugin marketplace update claude-plugins-official` (or `add anthropics/claude-plugins-official`), retry |
| Finder shows the repo folder as an application | Cosmetic: macOS treats any `*.app` directory as a bundle. Clone into a different local name — `gh repo clone pitscher/MicLine.app micline` ([file 02 §2.4](02-setup.md)); git and the remote are unaffected |
| Away for weeks/months, lost the thread | Run the re-entry protocol below — never "scan the repo for hours" |
| A spec keeps growing ("just one more thing") | New scope = new feature brief + new loop. Small loops are the point |
| Unsure whether a change needs the full loop | [file 05 §7.12](05-feature-loop.md): repo-tracked but behavior-free → chore-lane PR; GitHub-only → click + log line; Cloudflare-only → prefer wrangler.jsonc/CLI, else click + log line |

## Re-entry after a pause (days or months)

The repo is built so its own artifacts *are* the state — recovery is a short checklist, not archaeology:

0. **Open `docs/build-guide/PROGRESS.md` first** — its *Current position* line names exactly where you stopped (you set it in the session-end ritual, README "Tracking your progress"). The steps below then *verify* that claim against reality rather than reconstruct it from scratch.
1. **Orient outside-in:** the Releases page (= exactly what PRD runs) → the open Milestone + Roadmap Project (= what was planned) → open issues/PRs → `git log --oneline -15` (= what DEV has beyond the last release).
2. **Freshen tools:** run the [file 02 §2.9](02-setup.md) ritual — brew tools + Claude Code, `specify self check` / `self upgrade` (with the §2.5 re-init if it moved), and any Node/pnpm pin bumps via the chore lane.
3. **Re-verify the point-in-time facts** of [file 01](01-model-strategy.md) — model lineups and prices move; the routing *pattern* survives, the names may not.
4. **The re-entry prompt** — fresh session, strongest available model, `/effort high`:

   ```text
   Read docs/build-guide/PROGRESS.md, docs/product/milestone-map.md,
   docs/product/feature-inventory.md, the last 15 commits, all open issues
   and PRs, and any specs/⟨NNN⟩ directory whose branch is unmerged.
   Summarize: (1) what is live (latest release), (2) what sits on DEV but
   is unreleased, (3) any half-finished feature — propose /speckit.converge
   for it, (4) whether PROGRESS.md matches this reality (name mismatches),
   (5) the single most sensible next action. Change nothing.
   ```

5. **If a user bug report brought you back:** skip straight to the bugfix lane ([file 05 §7.11](05-feature-loop.md)) — issue, failing test, fix, PATCH release.

## Cheat sheet

| Phase | You run | Model / effort | Output | Gate |
|---|---|---|---|---|
| Setup | [file 02](02-setup.md) steps + [file 03](03-github-operations.md) read-through | — | tooling, guardrails, spec-kit, CLAUDE.md, env design | — |
| Brainstorm | P1 | Fable · max | product-brief, feature-inventory, decision-log | G1 |
| Tech context | P2 | Fable · high | constitution-input, tech-context, briefs | G2 |
| Constitution | P3 (`/speckit.constitution`) | Fable · max | constitution.md | G3 |
| Milestones | P4 | Fable · high | milestone-map, 4 epic briefs | G4 |
| Per feature | issue → P5a→P5g (`specify→clarify→plan→checklist→tasks→analyze→[issues]→implement→converge`) | file 01 table | specs/⟨NNN⟩/*, code, PR → auto-deploys DEV | G-A/B/C/D |
| Chore/config | [05 §7.12](05-feature-loop.md) lane: PR + `type:chore`, or UI click + manual-config-log line | Sonnet · low–med | small PR, or one log line | G-D if PR |
| Ship to PRD | `gh release create vX.Y.Z --generate-notes` → approve `production` env in the Actions run | you | Release notes = changelog; PRD deploy | approval click |
| Release | M1 gate list ([file 06](06-m1-plan.md)) | you | v1.0.0 live | — |
| M2–M5 | [file 07](07-m2-to-m5.md) kickoff ritual → loop | strongest · max for kickoff | new briefs → loops | per-loop |

## Assumptions log (flag if wrong)

1. Repo is **public**, single-repo, license **AGPL-3.0** (your decision, executed in [file 02 §2.3](02-setup.md)). Sole-author caveat: accepting outside contributions without a CLA freezes your relicensing freedom.
2. M1 feature cut = F000–F007 as in §8; pricing page dropped from M1 (M3 parked).
3. `users.role`, entitlement seam, audit log, soft-delete, structured logs, and centralized design tokens are pulled into M1 as forward-compat demands (per P4) even though their UIs land in M2/M4/M5.
4. Facts pinned as of **2026-07-04** (re-checked and toolchain-corrected 2026-07-05): Fable-on-Pro window ends July 7; Sonnet 5 is the Pro default; Spec Kit v0.12.x commands/flags as documented above (0.12.4 at verification); local toolchain = Node 24 (Active LTS) + pnpm 11, pinned in-repo via `.node-version`/`packageManager`, corepack-free; Cloudflare Email Service binding/behavior as summarized in P2. All are re-verified by the workflow itself (P5c doc-fetch rule) at use time.
5. i18n is first-class from F000 (Lingui, `en`+`de`, strings-through-macros enforced by the build); the translation *pipeline* (TMS) is a parked M2 candidate; the optional Lunaria status board ([file 06](06-m1-plan.md)) may precede it.
6. Execution order across milestones is M1 → M2 → M4 → M5 → M3-last-if-ever.
7. Every manual GitHub/Cloudflare click gets one line in `docs/runbooks/manual-config-log.md` — raw material for the M5 rebuild-from-zero runbook.
8. Repo is `github.com/pitscher/MicLine.app`, cloned locally as `micline` (Finder-bundle workaround).
9. DEV/PRD implemented as Wrangler environments (`--env dev` / `--env production`) with fully separate worker/D1/KV resources; shipping = publishing a GitHub Release ([file 03](03-github-operations.md)). The required-reviewer gate on `production` is free because the repo is public.
10. This guide is published in-repo at `docs/build-guide/` with the point-in-time disclaimer in its README.
11. CLAUDE.md only (no AGENTS.md); plugins limited to typescript-lsp + optional pr-review-toolkit ([file 08](08-claude-code-reference.md)).
12. Sequential development, one feature branch at a time, you review every gate. If that ever feels too slow, loosen G-A/G-C first — never G-B (plan) or G-D (merge).
13. Progress state lives **only** in `docs/build-guide/PROGRESS.md` (session-end ritual, README); the guide files themselves are never edited to track state.
14. Non-code changes follow the [file 05 §7.12](05-feature-loop.md) lanes; anything not expressible in the repo gets a `manual-config-log.md` line. Infrastructure is recreatable from the repo alone via wrangler.jsonc + F000's resource-bootstrap block + the runbook (no Terraform for now — [file 02 §2.8](02-setup.md)).


**Next:** [10-whats-next.md](10-whats-next.md) waits for you after the M1 gate; day to day, [05-feature-loop.md](05-feature-loop.md) is where life happens. **Carry forward:** after any pause, PROGRESS.md first, then the re-entry protocol; if an assumption is wrong, fix it at the matching gate.
---
[◀ Claude Code reference: context, cost, plugins](08-claude-code-reference.md) · [⌂ Index](README.md) · [What's next ▶](10-whats-next.md)
