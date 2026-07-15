> **MicLine Build Guide — Part 2 of 10** · [◀ Model strategy](01-model-strategy.md) · [⌂ Index](README.md) · [GitHub operations: environments, releases, lifecycle ▶](03-github-operations.md)

**In this part:** From bare MacBook to a fully bootstrapped repo: tooling, Claude Code on your Pro login, AGPL + guide + CONTRIBUTING committed, Spec Kit installed, CLAUDE.md written, the Cloudflare/GitHub-environment timeline, the Infrastructure-as-Code posture (§2.8), and the keep-the-toolchain-current ritual (§2.9) you'll re-run for the life of the project.
**Prerequisites:** File 01 read · Homebrew · GitHub account with the empty `pitscher/MicLine.app` repo · your Cloudflare account (`micline.app` zone already in it) · Claude Pro. · **Your time:** ~60–90 min today; the §2.7 rows execute later (≈30 min total) exactly when the timeline says. *(your active attention; Claude runtime and limit pauses excluded)*

## §2 Phase 0 — One-time setup (macOS)

Everything here assumes your MacBook Pro (Apple Silicon or Intel — identical steps) with [Homebrew](https://brew.sh) installed. One prerequisite Homebrew itself needs: the Xcode Command Line Tools (`xcode-select --install` — macOS prompts automatically on first `brew`/`git` use; includes git). Editor: anything you like; VS Code is a fine default (`brew install --cask visual-studio-code`) — Claude Code runs in the terminal regardless.

### 2.1 Local tooling

```bash
brew install fnm uv gh gitleaks pnpm   # node manager, spec-kit runtime, GitHub CLI, secret scanner, pnpm 11.x
fnm install 24 && fnm default 24       # Node 24 = current Active LTS (the even-numbered LTS line; 22 is in maintenance, 26 is "Current" and not LTS before Oct 2026)
gh auth login                          # authenticate the GitHub CLI (browser flow) — also handles git push auth
git config --global user.name "⟨your name⟩"
git config --global user.email "⟨github-noreply-or-real email⟩"   # noreply keeps your address out of public commits
```

Then add fnm's shell hook to `~/.zshrc` — **mandatory, not optional** (without it your shell never sees fnm's Node, and whatever else is on PATH silently wins — usually Homebrew's `node`, which brew's pnpm pulls in as a dependency and which tracks the newest non-LTS "Current" line, 26.x today):

```bash
echo 'eval "$(fnm env --use-on-cd)"' >> ~/.zshrc && exec zsh
```

`--use-on-cd` makes fnm switch Node per project: F000 commits a `.node-version` file, so entering the repo always activates the pinned version. pnpm works the same way from the other side: any installed pnpm ≥ 10 reads the exact version pinned in the repo's `package.json` `packageManager` field (also F000) and runs *that* — **the repo, not your machine, owns runtime versions**, which is what makes §2.9's "just upgrade the machine tools" safe.

Sanity check: `node -v` → v24.x · `pnpm -v` → 11.x · `uv --version` · `gh auth status` → logged in · `gitleaks version`. If `node -v` shows anything else, the fnm hook is missing or shadowed — `which -a node` shows the order ([file 09](09-troubleshooting.md) has the fix row).

> 💡 **Nice to know —** an earlier draft of this guide (and most tutorials) pinned pnpm via **corepack** (`corepack enable && corepack prepare pnpm@… --activate`). That path is dead on a current machine: corepack was only ever distributed with Node < 25, so Node 25/26 don't ship it — and Homebrew's `node` formula doesn't include it either — which is exactly why the command fails with "command not found". It's also unnecessary now: pnpm ≥ 10 manages its own version from the `packageManager` field, doing corepack's one job without corepack.

Do **not** install wrangler globally; it becomes a repo devDependency in F000 (run it as `pnpm wrangler …`). You need no Cloudflare account until the end of F000 (§2.7 has the exact timeline).

### 2.2 Claude Code

1. Install — either path gives the same self-contained native binary (avoid the `npm install -g @anthropic-ai/claude-code` route from older tutorials: it works, but ties updates to manual npm steps and buys nothing here; if it's present from before, `npm uninstall -g @anthropic-ai/claude-code` so it can't shadow the new binary on PATH):
   - **Native installer** (Anthropic's recommended default — auto-updates in the background): `curl -fsSL https://claude.ai/install.sh | bash`
   - **Homebrew cask**: `brew install --cask claude-code` — tracks the *stable* channel (≈1 week behind, skips releases with major regressions; `claude-code@latest` is the sibling cask on the fast channel). Brew installs don't self-update: `brew upgrade claude-code` is on you (§2.9), or set `CLAUDE_CODE_PACKAGE_MANAGER_AUTO_UPDATE=1` and Claude Code runs the brew upgrade for you.

   Verify with `claude --version` (`claude doctor` diagnoses install/auth/PATH issues; `claude update` triggers a manual update). Model minimums at verification: Sonnet 5 needs v2.1.197+, Opus 4.8 needs v2.1.154+ — mind that the **stable cask can sit below a days-old model's floor** (it was 2.1.193 on 2026-07-05, i.e. *no Sonnet 5*); if so, upgrade, switch to `claude-code@latest`, or use the native installer.
2. First run (`claude` in any directory): log in with your **Pro** account via the browser flow. Do **not** set `ANTHROPIC_API_KEY` in your shell — that would silently bill API pay-as-you-go instead of using your subscription.
3. Claude Code is directory-scoped: always `cd` into the repo first. The `/speckit.*` commands only exist inside the initialized repo (§2.5).
4. Run `/model` once and **write down which models your plan shows** (expect: Sonnet 5, Fable 5 until Jul 7, possibly Opus 4.x). This decides which column of the [file 01](01-model-strategy.md) routing table you live in.
5. Inside a session: `/config` — keep permission prompts **ON** (approving each file-write/command is your inline review surface during implement); optionally `/statusline` to permanently show model + context usage.

### 2.3 License — decision made: AGPL-3.0

You chose AGPL-3.0. It's the right fit for a public-repo SaaS: unlike GPLv3, it closes the network-use loophole — anyone who hosts a modified MicLine must publish their modifications, which is the strongest deterrent open source offers against clone-and-compete hosting. Two rules keep your future options open (not legal advice): (a) as **sole author** you may relicense or dual-license at any time — this freedom ends the moment you merge an outside contribution, so if PRs ever arrive, require a CLA or decline them; (b) the German operator duties you flagged (Impressum/DDG, VAT, AGB, Widerruf) attach to *running/selling the service*, not to the code license — they stay parked with M13 ([file 07 §9.9](07-m5-to-m13.md)).

Execution happens in the bootstrap commit below (the repo is empty, so you *add* the license rather than replace one).

### 2.4 Bootstrap the empty repo (incl. publishing this guide in it)

Clone into a different **local** folder name than the repo (`gh repo clone` takes one as its second argument) — the GitHub repo name stays `MicLine.app`, git doesn't care.

> 💡 **Nice to know —** the different local name exists because Finder renders any directory named `*.app` as an application bundle (that's LaunchServices doing its job, not damage). It's purely cosmetic; the terminal, git, and the remote are unaffected either way.

```bash
gh repo clone pitscher/MicLine.app micline && cd micline

# 1) License
curl -fsSL https://www.gnu.org/licenses/agpl-3.0.txt -o LICENSE

# 2) Skeleton
mkdir -p docs/inputs docs/product docs/engineering docs/build-guide
cat > .gitignore <<'EOF'
node_modules/
.dev.vars
.env*
.wrangler/
dist/
build/
.DS_Store
EOF

# 3) Publish this guide inside the repo
cp -R ⟨where-you-downloaded-it⟩/micline-build-guide/* docs/build-guide/
mv docs/build-guide/inputs/tech-decisions.md docs/inputs/tech-decisions.md

# 4) Optional: if you kept the original planning document, park it as extra input
#    (the workflow no longer requires it — tech-decisions.md carries the essentials)
# cp ⟨IMPLEMENTATION_SPEC⟩.md docs/inputs/legacy-spec.md

# 5) Minimal README (F000 replaces it with the real one + badges)
printf '# MicLine.app\n\nThe live question line for every event. **Work in progress** — see [docs/build-guide](docs/build-guide/README.md) for how this is being built.\n\nLicense: AGPL-3.0\n' > README.md

# 6) Ground rules — for humans and Claude alike (F000 links it from the real README)
cat > CONTRIBUTING.md <<'EOF'
# Contributing to MicLine.app

MicLine is built spec-first by a solo maintainer; see docs/build-guide/ for the full process.

- **One concern per PR.** No drive-by changes: if you touch it, it's because the PR's single purpose requires it.
- **Conventional commits** (feat:, fix:, docs:, chore:, …) — release notes are generated from them and from PR labels.
- **Spec-driven:** new behavior needs a spec under `specs/` before code. Bugfixes need a failing test first.
- **Issue first, PR second.** A PR without a linked, previously accepted issue may be closed unread. This is the anti-spam rule for humans and agents alike.
- **AI-assisted contributions are welcome — under human accountability.** Coding agents must follow the repo's agent context (CLAUDE.md today; AGENTS.md if present) and the spec-first rule above. The submitter reviews every line before opening the PR, discloses the tool/model via the PR template's AI-disclosure checkbox, and never opens bulk or automated PRs.
- **Translations:** a translation PR may only touch the catalog files of its own locale (`**/locales/⟨lang⟩/*`). Nothing else.
- **Outside contributions** currently require agreeing to a CLA before merge (sole-author licensing flexibility). Open an issue first; small PRs without prior discussion may be declined.
- **Security issues:** never via public issue — use GitHub's private vulnerability reporting (see SECURITY.md).
EOF

# 7) The manual-config log — start the habit NOW (this file later writes your
#    M12 "rebuild from zero" runbook for you; one line per manual click-step)
mkdir -p docs/runbooks
cat > docs/runbooks/manual-config-log.md <<'EOF'
# Manual configuration log

Every step performed by hand in a web UI (GitHub settings, Cloudflare dashboard) gets ONE line here,
newest last: `YYYY-MM-DD · system · exact path · what was set`. This is the raw material for the
future docs/runbooks/rebuild-from-zero.md (M12). If it's not logged, it can't be rebuilt.
EOF
```

> 💡 **Nice to know —** publishing the guide in-repo (the `cp -R` above) is a deliberate, examined choice, not laziness: it costs nothing you weren't already exposing (the repo will contain `specs/` and a public roadmap anyway), and it's a strong portfolio artifact — the repo documents not just the product but the *process*. Two caveats, both handled: parts are point-in-time (the guide's README carries a dated disclaimer for exactly this), and it must never tempt you to commit anything secret (it doesn't; it teaches the opposite).

Now the **GitHub-side switches** (repo → Settings, before first push — and log each in `docs/runbooks/manual-config-log.md`):

1. *Code security* → enable **Secret scanning**, **Push protection**, and **Private vulnerability reporting** (all free on public repos).
2. *General → Features* → enable **Issues** and **Discussions** (Discussions stays quiet until the M4 / `v0.4.0` public beta; enabling now costs nothing).
3. *Actions → General → Fork pull request workflows* → select the strictest approval option (**require approval for all outside collaborators**). Why: a stranger's PR must never run workflows unreviewed — and note that fork PRs can't read your secrets anyway, because those live in Environments.
4. Your **account** (not the repo): enable **2FA** on GitHub *and* Cloudflare now if you haven't — a hardware key or passkey ideally. Every control in this guide is downstream of these two accounts.
5. Optional "do not disturb": *Settings → Moderation options → Interaction limits* → **Limit to prior contributors** (max 6 months, renewable). Strangers then can't open issues/PRs/comments while you're heads-down building; lift it at launch. This is the honest answer to "how do I stop random people interfering" — a public repo can't forbid participation permanently, but branch protection means nobody but you can ever *merge*, templates + this limit keep the noise near zero, and [file 03 §A/§G](03-github-operations.md) covers the rest.

Local pre-commit gitleaks (throwaway; F000 replaces it with a committed, tool-managed version that survives fresh clones):

```bash
cat > .git/hooks/pre-commit <<'EOF'
#!/usr/bin/env bash
# scans staged changes; `gitleaks protect --staged` in older tutorials is the deprecated alias (since v8.19)
exec gitleaks git --pre-commit --staged --verbose
EOF
chmod +x .git/hooks/pre-commit
```

Commit and push: `git add -A && git commit -m "chore: bootstrap — AGPL-3.0, guide, guardrails" && git push`

### 2.5 Install Spec Kit

```bash
uv tool install specify-cli --from git+https://github.com/github/spec-kit.git@v0.12.4   # pin the latest release tag (v0.12.4 at verification — check the Releases page; the un-tagged branch tip is an unreleased dev build)
cd micline                   # if you aren't already inside the repo
specify init --here --force --integration claude
specify extension add git    # opt-in since v0.10.0 — installs the auto-branching the file-05 loop depends on
git add -A && git commit -m "chore: spec kit scaffolding" && git push
```

Notes:
- `--here --force` initializes into the existing non-empty repo (it warns about merging into a non-empty directory — expected, confirm).
- `--integration claude` is the current flag; the older `--ai claude` in most tutorials/videos was **removed in v0.10.0** and errors.
- The **git extension** is what makes `/speckit.specify` create the `⟨NNN⟩-⟨slug⟩` branch automatically; it stopped shipping by default in v0.10.0, hence the explicit `extension add` above.
- What you get: `.claude/commands/speckit.*.md` (the slash commands), `.specify/memory/constitution.md` (template), `.specify/templates/` (spec/plan/tasks templates — overridable per project via `.specify/templates/overrides/`), `.specify/scripts/bash/` (helpers).
- Verify inside Claude Code: launch `claude`, type `/speckit` — you should see `constitution, specify, clarify, plan, checklist, tasks, taskstoissues, analyze, implement, converge`.
- Spec Kit can alternatively install as Claude Code *skills* (`--integration-options="--skills"` → `.claude/skills/`). The commands mode above is the simpler, better-documented path — pick one mode, never both.
- Later upgrades are first-party now: `specify self check` (read-only "am I current?") → `specify self upgrade` (reinstalls the latest release tag in place) → back up `.specify/memory/constitution.md` → re-run `specify init --here --force --integration claude` → `git restore` your constitution if the re-init touched it → re-add the git extension if the re-init dropped it. §2.9 tells you when this runs.

### 2.6 Root `CLAUDE.md` (project memory)

Create this by hand (Spec Kit's `/speckit.plan` later appends stack context automatically — leave its markers alone when that happens). Keep it under ~150 lines forever; the reasoning and all context/cost discipline lives in [file 08 §B](08-claude-code-reference.md). One verified note up front: Claude Code does **not** read `AGENTS.md` — CLAUDE.md is the only memory file this project uses.

```markdown
# MicLine.app — agent context

## Non-negotiables (full text: .specify/memory/constitution.md)
- PUBLIC repo: never write a secret, token, key, or real email address into any file, log, or commit. Secrets live only in `.dev.vars` (gitignored), Wrangler secrets, and GitHub environment secrets.
- 100% Cloudflare for compute/storage/delivery. External HTTP APIs only where no Cloudflare primitive exists, and only behind a service-layer seam.
- Spec-driven: no feature code without a `specs/⟨NNN⟩-*/` spec+plan+tasks. If asked to code outside that flow, push back and point to the loop.

## Ground truth documents
- Product: docs/product/product-brief.md, docs/product/decision-log.md, docs/product/feature-inventory.md (authoritative for milestones M1–M13 + target versions)
- Engineering: docs/engineering/tech-context.md  ← read before ANY /speckit.plan or implementation work
- Process: docs/build-guide/ (how this project is built; current position: docs/build-guide/PROGRESS.md)
- Inputs (reference only): docs/inputs/

## Cloudflare documentation protocol (critical — operator's first Cloudflare project)
Before implementing against any Cloudflare product API, fetch current docs — never rely on training data:
- Product page index: https://developers.cloudflare.com/⟨product⟩/llms.txt  (workers, d1, durable-objects, kv, email-service, turnstile)
- Any docs page as Markdown: append `index.md` to its URL. All products: https://developers.cloudflare.com/llms.txt

## Commands
- pnpm dev · pnpm test · pnpm typecheck · pnpm lint · pnpm build   (exist after F000)
- Deploys happen ONLY via GitHub Actions (DEV on merge to main; PRD on published Release). Never run `wrangler deploy` or remote migrations locally.

## Conventions
- TypeScript strict; zod at every boundary; Drizzle for all SQL; no `any`.
- Conventional commits, small commits per task.
```

Commit it. Nested per-package CLAUDE.md files come later, in F000.

### 2.7 Cloudflare & GitHub-environment prerequisites — timeline (manual, ~30 min total)

Good news first: you already own `micline.app` and the zone is in your Cloudflare account — so there is **no zone/DNS migration step at all**, and even `dev.micline.app` needs no manual DNS: F000 configures both hostnames as Workers *custom domains*, and Cloudflare creates the records automatically on each environment's first successful deploy. Log every row below in `docs/runbooks/manual-config-log.md` as you do it.

| Row | When | What (exact paths) | Where |
|---|---|---|---|
| 1 | **End of F000** (its exit checklist tells you) | (a) **Workers Paid** ~$5/mo: Dashboard → *Compute (Workers & Pages)* → *Plans* → upgrade — required for production Durable Objects + Email Service sending. (b) **CI API token**: dash → profile icon → *My Profile* → *API Tokens* → *Create Token* → start from the "Edit Cloudflare Workers" template, then add permissions **D1: Edit** and **Workers KV Storage: Edit**; name it `micline-ci` (resource-naming rule, tech-context §3); copy it once. (c) **Account ID**: shown in the dashboard sidebar of any zone. (d) **GitHub Environments**: repo → *Settings* → *Environments* → create `dev` (no protection rules) and `production` (tick **Required reviewers**, add yourself) → inside **each**, add secrets `CLOUDFLARE_API_TOKEN` and `CLOUDFLARE_ACCOUNT_ID`. Environment-scoped, never repo-wide, never in the tree. | dash.cloudflare.com · GitHub → Settings → Environments |
| 2 | Before F002 (M2 auth) | **Email Service: onboard BOTH sending domains** — Dashboard → *Compute* → *Email Service* → *Email Sending* → *Onboard Domain* → `micline.app`, then repeat for `dev.micline.app` (domains onboard separately; each gets its own MX/SPF/DKIM/DMARC records on a `cf-bounce` subdomain + `_dmarc`; verification usually 5–15 min each). PRD sends from `@micline.app`, DEV from `@dev.micline.app` — DEV traffic never touches production sending reputation (tech-context §14, 2026-07-14 review). Log both onboardings. | dash.cloudflare.com |
| 3 | Before F006 (M4 bot defense) | **Turnstile**: create one widget listing both hostnames `micline.app` and `dev.micline.app`; note the site key (public — a wrangler var) and secret key (→ per-environment Wrangler secret + local `.dev.vars` only). Local dev uses Turnstile's documented **test keys**. | dash.cloudflare.com → Turnstile |
| 4 | `SESSION_SECRET` before F002 (M2) · `TURNSTILE_SECRET_KEY` before F006 (M4) | Runtime secrets, per environment: `pnpm wrangler secret put SESSION_SECRET --env dev` (and `--env production`), same for `TURNSTILE_SECRET_KEY` once row 3 is done. Generate values with `openssl rand -base64 32`; paste into the prompt, never into a file. | terminal |
| 5 | M6 (admin UI) — **choose one wall mode** (tech-context §11.4, AF-7) | **(a) Cloudflare Access — recommended:** Zero Trust org + Access application `micline-admin` covering `/admin*` on **both** hostnames, one-time PIN IdP, operator-email allowlist policy; then set the `ACCESS_TEAM_DOMAIN` + `ACCESS_APP_AUD` vars — configuring Access automatically disables the password path (Access always wins). **(b) Password quick-start:** `pnpm wrangler secret put ADMIN_PASSWORD --env dev` (and `--env production`; generate with `openssl rand -base64 32`) — **know the default before you rely on it: the admin session cookie is valid for 5 days** (committed default), checks are timing-safe, failures are throttled, and the admin UI will nudge you toward (a). Neither configured ⇒ `/admin` fails closed with setup instructions. | dash.cloudflare.com / terminal |

> 💡 **Nice to know —** Azure→Cloudflare translation for your mental model: Workers ≈ Azure Functions (consumption) + Front Door edge in one · **Durable Objects ≈ Durable Entities / Service Fabric reliable actors** (single-threaded, stateful, addressable — the key concept for F003) · D1 ≈ managed serverless SQLite ("small Azure SQL") · KV ≈ Table Storage/Redis-ish eventually-consistent cache · R2 ≈ Blob Storage · Queues ≈ Storage Queues.

### 2.8 Infrastructure as Code — the posture (your "rebuild from the repo alone" requirement)

The whole Cloudflare setup must be recreatable from nothing but this repository — say the account gets wiped. That requirement is met **without Terraform**, deliberately, through four layers:

1. **`wrangler.jsonc` is the IaC** for everything a Worker owns: both environments, worker names, custom-domain routes (which auto-create DNS), Durable Object classes + migrations, D1/KV **bindings**, vars. It's committed, reviewed, and applied by the deploy workflows — declarative config, versioned in git.
2. **Resource creation is two documented CLI commands per environment.** D1 databases and KV namespaces are *created* (not just bound) via `pnpm wrangler d1 create …` / `pnpm wrangler kv namespace create …`; the copy-pasteable commands live in the committed **`docs/runbooks/resource-bootstrap.md`** (a convention + command stub since the 2026-07-14 tech review; F000 completes it with the real commands and ids — [file 06](06-m1-to-m4-plan.md) F000 bundle 2), including where each returned id goes in `wrangler.jsonc` and the account's resource-naming (`micline` prefix) + Resource-Tagging (`project:micline`, `env:*`) conventions from tech-context §3. Recreate = run the block, paste ids, push.
3. **Secrets are never IaC** — by design (Constitution Article I). The rebuild runbook lists *which* secrets exist and the generation command (`openssl rand -base64 32`), never values.
4. **The residual console-only clicks are exactly the §2.7 rows** (Workers Paid, API token, email-domain onboarding, Turnstile widget, GitHub Environments, and — only if you pick the recommended Access mode for the M6 admin wall — the row-5 Zero Trust setup; the password quick-start needs no console step) — each logged in `manual-config-log.md`, which M12 compiles into `docs/runbooks/rebuild-from-zero.md` and optionally *proves* against a scratch account ([file 07 §9.8](07-m5-to-m13.md)).

> 💡 **Nice to know —** why not Terraform/Pulumi now? Cloudflare has providers for both, but for a solo Workers-first stack they'd add a state file to protect, drift between provider and Wrangler views of the same resources, and coverage gaps on the newest products — cost without benefit while the console-only surface is five logged rows. The decision has a re-open trigger: at M12, count the runbook's remaining manual lines; if it's grown beyond a handful, evaluate the providers **then**, against their current coverage ([file 10 §C](10-whats-next.md)). Until then, wrangler.jsonc + the bootstrap block + the runbook *are* the IaC.

### 2.9 Keeping the toolchain current — the recurring ritual (every milestone kickoff, and after any pause longer than a few weeks)

The split that makes this safe: **machine tools ride latest; anything the repo depends on is pinned *in the repo* and bumped deliberately.** An ambient `brew upgrade` can never change what the project builds with — only a reviewed commit can.

**Machine tools — just upgrade them:**

```bash
brew update && brew upgrade fnm uv gh gitleaks pnpm && brew cleanup
brew upgrade --cask claude-code    # skip if you used the native installer — it auto-updates (`claude doctor` shows the last attempt)
specify self check                 # if it reports a newer release: `specify self upgrade`, then the §2.5 re-init procedure
```

Upgrading brew's pnpm is always safe: inside the repo, pnpm runs whatever `packageManager` pins, regardless of what brew installed.

**Repo-pinned runtimes — bump via a chore-lane PR ([file 05 §7.12](05-feature-loop.md)):**

- **Node** — the pin is `.node-version` (committed in F000). Check which line is Active LTS at nodejs.org/en/about/previous-releases (the even-numbered one; next handover: 26 becomes LTS in Oct 2026). To bump: edit `.node-version` in a PR, run `fnm install ⟨new⟩` locally, let CI prove it (CI reads the same file), merge. After a Node **major** switch, re-install per-version npm globals — under fnm these live inside each Node version (currently just F000's typescript-lsp prerequisite: `npm i -g typescript-language-server typescript`).
- **pnpm** — the pin is `"packageManager"` in the root `package.json`. Bump it in a PR (read the release notes on a **major** — pnpm majors change behavior). Dependabot doesn't touch this field, so it's on this ritual.
- **Everything else in the repo** (npm dependencies, GitHub Actions) is already Dependabot's job ([file 03 §A](03-github-operations.md)) — grouped weekly PRs through the normal chore lane.

**Point-in-time facts** (model lineups, prices, Spec Kit flags, Cloudflare beta behavior) need no ritual of their own: the workflow re-verifies them at use time (the P5c doc-fetch rule), and the milestone-start guide-freshness sweep ([file 10 §D](10-whats-next.md)) catches the rest — the README disclaimer lists what to look at.


**Next:** read [03-github-operations.md](03-github-operations.md) once, before any prompting. **Carry forward:** the manual-config-log habit; never set `ANTHROPIC_API_KEY`.
---
[◀ Model strategy](01-model-strategy.md) · [⌂ Index](README.md) · [GitHub operations: environments, releases, lifecycle ▶](03-github-operations.md)
