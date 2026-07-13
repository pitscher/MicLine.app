# Manual configuration log

Every step performed by hand in a web UI (GitHub settings, Cloudflare dashboard) gets ONE line here,
newest last: `YYYY-MM-DD · system · exact path · what was set`. This is the raw material for the
future docs/runbooks/rebuild-from-zero.md (M12). If it's not logged, it can't be rebuilt.

`2026-07-05 · GitHub repo · Settings → Advanced Security → Secret Protection · click "Enable" (enabled by default)`

`2026-07-05 · GitHub repo · Settings → Advanced Security → Push Protection · click "Enable" (enabled by default)`

`2026-07-05 · GitHub repo · Settings → Advanced Security → Private Vulnerability Reporting · click "Enable"`

`2026-07-05 · GitHub repo · Settings → General → Features → Issues · Check the "Issues" box (checked by default)`

`2026-07-05 · GitHub repo · Settings → General → Features → Issues → Issue Permissions · Select Creation allowed by: Collaborators only`

`2026-07-05 · GitHub repo · Settings → General → Features → Discussions · Check the "Discussions" box`

`2026-07-05 · GitHub repo · Settings → Actions → General → Approval for running fork pull request workflows from contributors · Select "Require approval for all external contributors" radio button + click "Save"`

`2026-07-05 · GitHub repo · Settings → Moderation options → Interaction limits → Limit to prior contributors · click "Enable" and select the option "6 months"`
