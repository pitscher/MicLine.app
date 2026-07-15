# MicLine.app — agent context

## Non-negotiables (full text: .specify/memory/constitution.md)
- PUBLIC repo: never write a secret, token, key, or real email address into any file, log, or commit. Secrets live only in `.dev.vars` (gitignored), Wrangler secrets, and GitHub environment secrets.
- 100% Cloudflare for compute/storage/delivery. External HTTP APIs only where no Cloudflare primitive exists, and only behind a service-layer seam.
- Spec-driven: no feature code without a `specs/⟨NNN⟩-*/` spec+plan+tasks. If asked to code outside that flow, push back and point to the loop.

## Ground truth documents
- Map: docs/README.md — where every document lives + which one is authoritative; check here first when unsure (keep it updated when docs move)
- Product: docs/product/product-brief.md, docs/product/decision-log.md, docs/product/feature-inventory.md (authoritative for milestones M1–M13 + target versions)
- Engineering: docs/engineering/tech-context.md  ← read before ANY /speckit.plan or implementation work
- Process: docs/build-guide/ (how this project is built; current position: docs/build-guide/PROGRESS.md)
- Ops: docs/runbooks/ (resource-bootstrap, security-incident-response, maintenance, manual-config-log; config-operations arrives with F008) · run costs: docs/engineering/runcost-estimate.md
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