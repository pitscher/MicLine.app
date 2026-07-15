# MicLine.app — Running-cost estimate (typical scenarios)

> Provenance: **post-interview tech review, 2026-07-14** (operator request). Status: **pre-implementation estimate from list prices** — all prices verified against Cloudflare docs on 2026-07-14; no measured data exists yet. The **M12 cost model (OPS-02)** supersedes this file with formulas from *measured* usage and feeds the admin € projection; until then, this is the operator's orientation. Currency: Cloudflare bills in USD; EUR figures assume **1 USD ≈ €0.90** — re-check the rate when reading. Re-verify prices at every milestone kickoff (build-guide 02 §2.9 · `docs/runbooks/maintenance.md`).

## The shape of MicLine's bill (read this first)

- **One fixed line dominates:** Workers Paid, **$5/month ≈ €4.50** for the account. It unlocks production Durable Objects, D1/KV scale, and Email Service sending. Everything else is metered **above generous included allotments** that MicLine's design envelope (~500 concurrent participants, one busy event at a time) does not come close to exhausting.
- Two structural cost-savers in the chosen architecture: **WebSocket Hibernation** (a DO bills duration only while awake; the runtime answers keep-alive pings without waking it) and **outgoing WS messages are free** — broadcasting the line to a full room costs nothing; only *incoming* messages bill, at a 20:1 ratio as DO requests.
- The **3-day content purge**, event auto-end, and account inactivity purge keep stored data near zero — storage lines never accumulate.
- The Cloudflare account is shared with other projects: MicLine's usage is attributable via the `micline` name prefix + Resource Tags (`project:micline`, tech-context §3), but the invoice itself is account-level.

## Price sheet (Workers Paid — verified 2026-07-14)

| Line item | Included per month | Overage |
|---|---|---|
| Workers requests | 10 M | $0.30 / M |
| Workers CPU time | 30 M CPU-ms | $0.02 / M CPU-ms |
| DO requests (HTTP, WS connects, alarms; incoming WS messages count 1/20) | 1 M | $0.15 / M |
| DO duration (awake time; 128 MB DO = 0.125 GB × seconds) | 400,000 GB-s | $12.50 / M GB-s |
| DO SQLite storage (billing live since 2026-01) | 25 B rows read · 50 M rows written · 5 GB-mo | $0.001 / M read · $1.00 / M written · $0.20 / GB-mo |
| D1 | 25 B rows read · 50 M rows written · 5 GB | $0.001 / M read · $1.00 / M written · $0.75 / GB-mo |
| KV | 10 M reads · 1 M writes/deletes · 1 GB | $0.50 / M reads · $5.00 / M writes · $0.50 / GB-mo |
| Email Service (sending) | 3,000 sends | $0.35 / 1,000 sends (new accounts also carry conservative auto-scaling daily quotas) |
| R2 (M7 emergency exports only) | — | $0.015 / GB-mo · Class A $4.50 / M · Class B $0.36 / M |
| Turnstile · Access (1 seat) · GitHub Actions/CodeQL (public repo) · zone/DNS | free at MicLine's scale | — |

## Scenarios

### S0 — Quiet month (both environments deployed, a handful of small events)

Every metered line sits far inside its allotment. **Total: ≈ €4.50** — the base fee is the whole bill. (Plus the pre-existing domain renewal, ~€10–20/year, outside this budget.)

### S1 — One fully booked baseline event (100 concurrent participants, 2 h, ~80 questions)

| Driver | Envelope estimate | Marginal cost if metered |
|---|---|---|
| Worker requests (SSR joins, API, reconnects) | ~5 k | ~$0.002 |
| DO requests (~150 connects + ~600 mutations/alarms + ~5 k incoming msgs ÷ 20) | ~1 k | ~$0.0002 |
| DO duration (worst case: awake the full 2 h) | 900 GB-s | ~$0.011 |
| Rows read/written (DO + D1), KV reads | thousands | ~$0.001 |
| Email (moderator login) | ~1 send | ~$0 |

**Marginal cost ≈ €0.01–0.02 — a fully booked event costs about one euro cent**, and while the monthly allotments aren't exhausted it is effectively €0.

### S2 — Ceiling event (1,000 concurrent presence, 3 h, M9-era with upvotes)

Worker requests ~30 k · DO: ~1.5 k connects + ~100 k incoming messages ÷ 20 ≈ 5 k request-equivalents · duration awake 3 h = 1,350 GB-s (~$0.017) · tens of thousands of row writes (votes, questions) — all inside allotments. **Marginal cost ≈ €0.03–0.05.**

### S3 — Busy month (the ~€20 target check)

40 × S1 events + 2 × S2 events + ~500 moderator logins + feedback mails: ~0.5 M Worker requests, ~40 k GB-s DO duration, ~600 emails — **every line inside its included allotment. Total: ≈ €4.50.** The product brief's ~€20 busy-month target has **≳4× headroom above the base fee alone**, and holds even if every metered estimate here is off by an order of magnitude.

### S4 — Abuse / spike sensitivity (what a bad actor costs before the brakes bite)

| Attack shape | Cost sensitivity | Brake |
|---|---|---|
| +1 M junk Worker requests | ≈ €0.27 | Cloudflare DDoS front door, per-IP ceilings (F006), WAF (runbook) |
| +1 M junk DO request-equivalents | ≈ €0.14 | per-IP room-scale ceilings, per-DO soft throttle |
| One DO held awake 24/7 | ≈ 324 k GB-s/mo — still inside the allotment | hibernation + event auto-end + purge |
| **10 DOs held awake 24/7** | **≈ +€25/mo** | the same — this is *why* hibernation correctness matters |
| Junk email sends (hypothetically 100 k) | ≈ €31 — the most expensive lever | account auto-scaling quotas cap this first; **per-email cap (F002)**, Turnstile (M4), creation kill-switch (M1), lockdown (M8) |

The alert-only budget threshold (notify, never auto-flip) arrives with M12; lockdown is the standing cost brake.

## What to watch (the honest list)

1. **Email overage past 3,000 sends/month** — the first line that grows with real adoption (every magic-link login = 1 send). At 10 k sends/month the bill is still only ≈ €2.20 extra.
2. **DO duration, if hibernation is accidentally defeated** — an app-level ping loop or a socket usage bug that keeps DOs awake multiplies the duration line silently. Glance at the usage dashboard on the `docs/runbooks/maintenance.md` cadence.
3. **Price changes** — this sheet is list prices as of 2026-07-14; the maintenance runbook re-verifies them.

**Caveats:** envelope math, not measurement; assumes the tech-context architecture (hibernation, snapshot+deltas, purge machinery) is implemented as designed. M12 OPS-02 replaces this file's numbers with measured formulas.
