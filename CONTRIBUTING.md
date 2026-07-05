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
