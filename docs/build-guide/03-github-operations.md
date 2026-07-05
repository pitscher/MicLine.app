> **MicLine Build Guide — Part 3 of 10** · [◀ One-time setup (macOS)](02-setup.md) · [⌂ Index](README.md) · [Product definition ▶](04-product-definition.md)

**In this part:** The operating model everything else plugs into — DEV/PRD environments, the five quality gates, Releases as changelog + ship-button, public visibility, moderation, pipeline security, and the twelve-step idea→PRD lifecycle. F000 later *implements* this design.
**Prerequisites:** File 02 done through §2.5 (repo + Spec Kit exist). · **Your time:** ~25 min careful read now; the setup blocks inside are executed later, where other files point at them. *(your active attention; Claude runtime and limit pauses excluded)*

## GitHub operations — environments, quality gates, releases, and the full feature lifecycle

This file is the **design** that F000 implements and that every later feature flows through. Read it once now; return to §F whenever you forget how a change reaches users.

> 💡 **Nice to know —** a pleasant fact runs through everything here: because the repo is public, GitHub gives you for free several capabilities that private repos pay for — unlimited Actions minutes on standard runners, CodeQL code scanning, and (the big one) **Environments with protection rules**, which is what makes a one-person PRD approval gate possible at all.

### A · The GitHub feature matrix (solo dev, public repo)

| Feature | Use it for | Set up | Verdict |
|---|---|---|---|
| **Issues + PR templates** | Every feature/bug starts life as an issue (feature_request.yml, bug_report.yml, config.yml pointing questions to Discussions, blank issues off); the PR template enforces purpose + `Closes #issue` + self-review checklist + an **AI-assisted change** disclosure checkbox | F000 | ✅ core |
| **Labels** | `type:feature` `type:fix` `type:docs` `type:chore` `type:security` + `area:*` — they drive auto release notes (§D) | after Gate 4 (block below) | ✅ core |
| **Milestones** | M1–M5 as GitHub Milestones; every issue/PR assigned to one → progress bars for free | after Gate 4 (block below) | ✅ core |
| **Projects (v2)** | ONE public board "MicLine Roadmap", grouped by Milestone, status Planned/In progress/Shipped → the public "what's planned". Turn on its **built-in workflows** (auto-add new issues, auto-set Done when the issue closes) so the board maintains itself — you never move cards | after Gate 4, via web UI (2 min), link in README | ✅ core |
| **Releases** | Versioned changelog + the PRD deploy trigger (§D) | F000 config, first used at M1 gate | ✅ core |
| **Actions** | CI quality gates + both deploy pipelines (§C) — free & unlimited on public standard runners | F000 | ✅ core |
| **Environments** (`dev`, `production`) | Scoped secrets + required-reviewer approval on `production` — free on public repos | [file 02 §2.7](02-setup.md) row 1 | ✅ core |
| **Branch protection / rulesets** | main requires PR + the five green checks + linear history; no force-push/deletion | end of F000 | ✅ core |
| **Dependabot** | npm + actions updates, grouped weekly so solo-you isn't spammed | F000 | ✅ core |
| **CodeQL** (default setup) | Static security analysis, free on public repos | F000 | ✅ core |
| **Secret scanning + push protection + private vulnerability reporting** | The public-repo safety net + a responsible-disclosure channel (pairs with `SECURITY.md`) | §2.4 | ✅ core |
| **Discussions** | User-facing Q&A/feedback/announcements once live — keeps Issues for actual work | enabled in §2.4, activated at M1 launch | ✅ core |
| **Deployments view** | Free byproduct of Environments: repo sidebar shows exactly what's on DEV/PRD right now | automatic | ✅ free |
| **README badges** | CI status, latest release, license — public health signals | F000 | ✅ |
| **CONTRIBUTING.md** | The repo's ground rules for humans *and* Claude (one concern per PR, conventional commits, translation-PR rule, CLA note) — created in bootstrap | [file 02 §2.4](02-setup.md) | ✅ core |
| **Moderation: interaction limits** | The public-repo "do not disturb" switch — restrict issues/PRs/comments to prior contributors for up to 6 months (renewable) while you build; lift at launch | Settings → Moderation options ([file 02 §2.4](02-setup.md) step 5) | ◻︎ situational |
| **Actions fork-PR approval** | Outside contributors' workflow runs wait for your click; fork PRs can never read secrets anyway (Environment-scoped) | 02 §2.4 step 3 | ✅ core |
| CODEOWNERS | Review routing — pointless with one human | — | ✖ skip |
| Wiki | Docs already live versioned in `docs/` | — | ✖ skip |
| GitHub Pages | For the *product*: skip — micline.app is the site. For internal tooling dashboards (the optional Lunaria translation board, [file 06](06-m1-plan.md)): fine | optional loop | ✖ product · ◻︎ tooling |
| Sponsors | Revisit if the repo gains an audience | — | ◻︎ later |

Milestone/label setup block (run once, after Gate 4 — [file 04](04-product-definition.md) points here):

```bash
for m in "M1 Core product" "M2 Stand-out + operations" "M3 Monetization (parked)" "M4 UI overhaul" "M5 Longevity"; do
  gh api repos/{owner}/{repo}/milestones -f title="$m"; done
for l in "type:feature#0e8a16" "type:fix#d73a4a" "type:docs#0075ca" "type:chore#cfd3d7" "type:security#b60205"; do
  gh label create "${l%%#*}" --color "${l##*#}" --force; done
```

The five titles above are the guide's working set — if Gate 4 renamed or re-cut milestones, substitute the titles from your `milestone-map.md` here.

**Agent-ready by construction.** You asked how the repo stays safe for AI agents — yours today, possibly others' later — without them spamming it or contributing unstructured. The answer is that the controls above already *are* the agent guardrails; no separate "AI policy" layer is needed: templates + blank-issues-off shape every entry point; **branch protection means no agent (or human) but you can ever merge**; fork-workflow approval means a stranger's agent can't even run CI without your click; CONTRIBUTING's issue-first, one-concern, and AI-disclosure rules ([file 02 §2.4](02-setup.md)) bind agents through their operators; CLAUDE.md + the constitution's Article VIII channel any spec-respecting agent into the [file 05](05-feature-loop.md) loop; gitleaks + push protection catch what any of them leak. A worked reference for this "AI-first repo" shape is **EmDash** (github.com/emdash-cms/emdash): AGENTS.md as the agent source of truth with CLAUDE.md symlinked to it, agent-readable commands, and a PR template whose AI-disclosure checkbox we adopt in F000. While you're solo, plain CLAUDE.md is the right call ([file 08 §B](08-claude-code-reference.md)); the day you open the repo, run the checklist in [file 10 §D](10-whats-next.md) — including the EmDash symlink move.

### B · Environment architecture (DEV / PRD)

One Cloudflare account, two fully separated deployments, defined as **Wrangler environments** in `wrangler.jsonc`:

| | `env.dev` | `env.production` |
|---|---|---|
| Worker name | `micline-dev` | `micline` |
| URL | `dev.micline.app` | `micline.app` |
| D1 database | `micline-dev` (own id) | `micline` (own id) |
| KV namespace | own id | own id |
| Durable Objects | isolated automatically (DOs live inside their Worker) | isolated |
| Secrets | `wrangler secret put ⟨X⟩ --env dev` | `--env production` |
| Deployed by | `deploy-dev.yml`: every merge to `main` | `deploy-prd.yml`: every **published Release**, after your approval |
| GitHub Environment | `dev` (no protection) | `production` — **required reviewer: you** |
| Purpose | Latest `main`, always; your smoke-testing ground; may break briefly | What users trust; changes only deliberately, in versioned increments |

Consequences worth internalizing: `main` **is** DEV — merging a PR ships it to dev.micline.app within minutes, so "it works on DEV" is a real claim, not "works on my machine". PRD lags DEV by exactly the set of merges since the last Release, which means `main` must stay releasable at all times — the quality gates below are what make that true, and the simple hotfix path (fix → main → verify on DEV → publish a PATCH release) follows from it. If you ever land something on `main` you're not ready to ship alongside a needed hotfix, the escape hatch is a release branch cut from the last tag — but with small features and honest gates you should never need it. Note also that this entire table is expressible from the repo alone: `wrangler.jsonc` plus F000's resource-bootstrap block — the IaC posture of [file 02 §2.8](02-setup.md).

### C · Quality gates (the five required checks)

`ci.yml` runs on every PR and push to main; branch protection makes all five **required**: `lint` (Biome), `typecheck` (tsc -b), `test` (Vitest in workers pool — this is where the F003 state-machine suite lives), `build` (react-router build), `secrets` (gitleaks over the diff). Plus continuous background gates that don't block merges: CodeQL, Dependabot, secret-scanning/push-protection. Your human gates (A–D) from [file 05](05-feature-loop.md) sit on top — the automated five catch regressions; you catch wrong direction.

### D · Versioning & releases — your question answered

**Yes — GitHub Releases with semver tags is the right mechanism**, and better than the alternatives for your goals, because one primitive covers four needs at once: (1) an immutable, human-readable **changelog** at `github.com/…/releases` — exactly your "people can check what current features are available"; (2) the **PRD deploy trigger** — publishing a release *is* shipping, so the changelog can never lie about what's deployed; (3) **auto-generated notes** — `.github/release.yml` categorizes merged PRs by their `type:*` labels, so notes write themselves and you only humanize the summary; (4) **traceability** — every prod incident maps to "which release", every release to its PRs, every PR to its spec and issue. (What it does *not* cover is "what's planned" — that's the public Project board + Milestones from §A. Together they answer both halves of your requirement.)

Semver policy for an app (simpler than library semver): **MAJOR** = a milestone completes or users must relearn something (M2 done → `v2.0.0`) · **MINOR** = new user-visible feature(s) (`v1.1.0`) · **PATCH** = fixes only (`v1.0.1`). Release cadence is yours: batch several merged features into one MINOR, or ship each — DEV absorbs merges either way; PRD moves only on publish.

`.github/release.yml` (F000 creates it):

```yaml
changelog:
  categories:
    - title: "✨ New"
      labels: ["type:feature"]
    - title: "🛠 Fixes"
      labels: ["type:fix"]
    - title: "🔐 Security"
      labels: ["type:security"]
    - title: "🧹 Internal"
      labels: ["type:chore", "type:docs"]
```

Cutting a release, end to end: `gh release create v1.1.0 --generate-notes --title "…"` → GitHub drafts categorized notes → you edit in a 2-line human summary → **Publish** → `deploy-prd.yml` starts → the run pauses on the `production` environment → Actions → *Review pending deployments* → approve → migrate + deploy run → the Deployments sidebar and the release page now agree about reality.

**Releasing without disturbing live events (zero-downtime, by design).** Workers deploys swap code at the edge without dropping in-flight HTTP requests, so page loads and API calls never notice a release. The one real seam is **WebSockets**: a production deploy restarts the Durable Objects, so every open socket drops once and reconnects — which is exactly why F004/F005 mandate auto-reconnect + fresh-snapshot logic. That logic is a *release-safety feature*, not just venue-Wi-Fi polish; and because each event's state lives in the DO's SQLite storage, it survives the restart — nobody loses their place in line, participants see at worst a one-second "reconnecting" flicker. Two rules keep this true forever: (1) **migrations are expand-first** — add columns/tables, ship code that tolerates both shapes, remove old shapes in a later release; never a schema change that code-from-one-release-ago can't read, because clients reconnect into new code mid-event. (2) **The approval pause is your timing gate**: before clicking approve, glance at whether a big event is live (an M2 admin metric; until then, judgment + quiet hours) and wait an hour if so. A *hard* "block release while anyone is online" gate is deliberately **not** built: events can run at any hour worldwide, so such a gate degrades into either "never ship" or a habit of overriding it — the human pause + reconnect-safe design is strictly better. If you ever outgrow this, Workers supports **gradual version rollouts** (percentage-based canary) — file that under M5, not now.

### E · Public visibility (what users and visitors see)

README top: badges (CI · latest release · AGPL-3.0) + three links: **Roadmap** (the public Project), **Changelog** (Releases), **Feedback** (Discussions). F007's marketing site footer links the same three. Discussions carries announcements (post each release) and user questions; Issues stays for confirmed work items — the issue templates' `config.yml` routes "questions" to Discussions automatically. Net effect: a stranger can, without asking you anything, see what MicLine does today (Releases), what's coming (Project/Milestones), and where to talk to you (Discussions) — your trust requirement, satisfied by linking three built-ins.

### F · The full lifecycle — from idea to users (the process you asked to understand)

```
 idea ──▶ ISSUE (template, milestone) ──▶ feature brief ──▶ Spec Kit loop (file 05)
                                                            specify→clarify→plan→tasks
                                                            →analyze→implement→converge
                                                                       │
                        ┌── you: GATE D review ◀── PULL REQUEST ◀──────┘
                        ▼            (5 auto checks must be green)
                     MERGE to main
                        │  deploy-dev.yml (auto)
                        ▼
                  DEV  dev.micline.app  ── you: smoke test
                        │
                        │  …more features merge; DEV accumulates…
                        ▼
              gh release create vX.Y.Z (notes auto-generated from PR labels)
                        │  publish ──▶ deploy-prd.yml ──▶ ⏸ waits for YOU
                        ▼                                   (approve `production`)
                  PRD  micline.app  ── you: smoke test
                        │
                        ▼
        announce in Discussions · Release page = changelog · issue auto-closed
```

In words, the twelve steps: (1) an idea becomes an **issue** on a milestone; (2) if it's new scope, it gets a **feature brief**; (3) the **Spec Kit loop** turns brief → spec → plan → tasks → code on a feature branch, with your gates A–C along the way; (4) a **PR** opens, `Closes #issue`; (5) the **five checks** run; (6) **you review** (Gate D); (7) **merge** — deploy-dev fires; (8) you **smoke DEV**; (9) when a user-worthy increment exists, you **publish a Release** — notes generate themselves from labels; (10) deploy-prd **waits for your approval**, you click it; (11) you **smoke PRD**; (12) the Release page updates the public changelog, the Project board moves cards to Shipped, and a Discussions post announces it. Every step leaves an artifact; nothing reaches users without passing five machines and one human — you. One boundary worth naming: this train is for changes that ship *behavior*. A change with little or no app code — a workflow tweak, a GitHub setting, a Cloudflare dashboard switch — takes one of the lighter lanes defined in [file 05 §7.12](05-feature-loop.md) instead.


### G · Security hardening addendum (repo & pipeline)

The application-side controls live where they're built (auth in F002, WS origin checks in F003, headers in F000, abuse in F006 — see the threat map below). What F000 must additionally implement on the **pipeline** side, because a public repo's CI is itself an attack surface: (1) **pin third-party Actions to full commit SHAs**, not tags — a hijacked tag then can't reach you, and Dependabot keeps the pins fresh; (2) every workflow declares top-level `permissions: contents: read`, jobs escalate only what they individually need — a compromised step then holds a near-useless token; (3) **never interpolate untrusted text** (issue titles, PR bodies, branch names) into `run:` blocks — classic template-injection; our workflows don't template user input at all; (4) secrets exist **only as Environment secrets**, so fork PRs can't read them and prod secrets additionally sit behind your reviewer gate; (5) CodeQL, gitleaks (hook + CI), and push protection run as standing tripwires. 

The one-screen threat map (threat → controls → built where):

| Threat | Controls | Where |
|---|---|---|
| Secret leaks into public tree | pre-commit gitleaks · push protection · CI `secrets` job · Environment-scoped secrets · `.dev.vars` gitignored | 02 / F000 |
| Compromised third-party Action | SHA pinning · least-privilege `permissions` · Dependabot on actions | F000 |
| Hostile PR / issue spam | fork-workflow approval · branch protection (only you merge) · templates + blank-issues off · interaction limits | 02 / 03 §A |
| Your account taken over | 2FA/passkeys on GitHub **and** Cloudflare (02 §2.4 step 4) | you |
| Magic-link auth abuse | hashed single-use tokens (TTL 15 min) · anti-enumeration 200s · rate limits · Turnstile | F002 / F006 |
| Session theft / forgery | HMAC-signed HttpOnly+Secure+SameSite=Lax cookies · constant-time compare | F002 |
| WebSocket hijack / cross-site use | Origin allowlist at upgrade · DO re-checks role on every mutating message | F003 |
| Injection (SQLi / XSS) | Drizzle parameterized SQL · zod at every boundary · React escaping · security headers incl. CSP | F000 / F007 |
| DoS & cost attacks | Cloudflare front door · Turnstile · rate-limit matrix · per-DO soft throttle | F006 |
| Vulnerable dependencies | minimal deps (Article III) · Dependabot weekly · CodeQL | F000 |

Honest residual risks, so you know what you're accepting: a public repo means attackers read your exact code (mitigated by having no secrets and no security-through-obscurity — everything above assumes the source is known); and beta Email Service means deliverability/limits could shift (mitigated by the service seam). Nothing here is "trust me" — every row has a check you can point at.

**Next:** [04-product-definition.md](04-product-definition.md) — the Fable-window work. **Carry forward:** `main`==DEV; shipping==publishing a Release; log every manual click.
---
[◀ One-time setup (macOS)](02-setup.md) · [⌂ Index](README.md) · [Product definition ▶](04-product-definition.md)
