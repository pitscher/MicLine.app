# F007 — Marketing site & legal pages

**Milestone:** M4 — Public-beta readiness · **Target version:** `v0.4.0` · **Implements:** MKT-01, MKT-02
**Depends on:** F005 (join runtime the code box targets), F010/F011 semantics (the privacy page must describe enforced reality). Light ceremony; Sonnet-class per file 06.

## User value

Moderators land, understand the line pillar in one screen, and create an event; lost participants get rescued to their event via the prominent code box; and every privacy/retention claim on the site is **true because the machinery behind it already shipped** — trust as a feature.

## In scope

- **Landing (MKT-01):** moderator-pitch hero on the line pillar, 3-step how-it-works, create CTA, prominent join-code box, honest privacy block, visual line mock; clean — depth in sub-pages; **no pricing page, no repo link, no SLA disclaimers on the hero**.
- **Sub-pages (MKT-02):** how it works · FAQ (incl. the honest outage entry pointing at the emergency machinery) · privacy — truthful lifetimes and deletion incl. the **event-row vs event-content split** and the **≤30-day infrastructure point-in-time-recovery sentence** (interview decision), export responsibility, geolocation disclosure · terms — best-effort/no-SLA phrased as what MicLine *does* · Impressum · contact.
- **SEO (AF-5, binding checklist):** per-locale titles + meta descriptions, canonical URLs, hreflang (en/de), OG/Twitter cards, JSON-LD structured data (WebSite/Organization + FAQPage), semantic heading structure; `robots.txt` + `sitemap.xml` covering **marketing pages only**; static SSR; page copy through Lingui (en+de); no client JS beyond CTA + locale switcher; **no trackers**.
- **Indexing allowlist (AF-6):** flips exactly the marketing surfaces out of F000's default-deny `X-Robots-Tag: noindex` baseline — **on production only**; DEV and every non-marketing route (join, board, console, auth, admin, exports) stay noindexed forever.
- Footer: GitHub repo, Releases (changelog), roadmap Project, Discussions.

## Out of scope

Visual identity overhaul (M11) · pricing/billing anything (M13, parked) · counsel-reviewed legal text (explicitly acknowledged as shipping without counsel until M13 — FR-6).

## Dependencies & seams

The no-SLA placement rules (product brief "Operations posture") are binding copy constraints. Privacy-page wording items owned here: PITR honesty sentence, "don't put personal data in titles" guidance.
