# AI Builder and Hosting Strategy

## Decision

Use **Cursor as the primary engineering environment**, **GitHub as the source of truth**, and **Vercel for the web/control-plane deployment surface**. Do not use an AI app builder as the sole authority for financial services code.

### Why Cursor

This project needs repository-wide refactoring, typed contracts, database migrations, event schemas, provider adapters, tests, security review and controlled CI/CD. Cursor is positioned as an engineering IDE/agent with MCP, skills, cloud agents and team controls. Current published pricing: Pro $20/month, Pro+ $60/month, Ultra $200/month, Teams Standard $40/user/month, with Enterprise custom. Verify pricing before purchase.

### Why not make Lovable/Bolt/Replit the primary backend authority

- Lovable is strong for rapid product UI and full-stack prototypes, GitHub sync and enterprise governance, but the financial core should still be reviewed and controlled in the repository.
- Bolt is excellent for rapid full-stack prototypes and has current Pro/Teams options, but token-oriented generation and browser-first workflows are better suited to acceleration than financial-system source-of-truth governance.
- Replit is useful for collaborative agentic development and hosted prototypes, but the target architecture needs explicit control over deployment, data, provider secrets, network boundaries and production CI/CD.

These tools can be used as secondary prototyping tools if desired. They are not the production authority.

## Hosting model

```text
GitHub
  -> CI/CD
  -> Vercel web/control plane
  -> managed Postgres/PostGIS
  -> Redis
  -> durable queue/event bus
  -> worker/runtime for ingestion and reconciliation
  -> object storage
  -> Stripe / GoCardless / commerce / 3PL adapters
```

Vercel is appropriate for the web/API edge and control-plane experience. Heavy ERP ingestion, reconciliation, long-running consumers and Kafka-style processing should run in a worker/container environment rather than assuming a request/response serverless function can perform all work.

## Recommended build mode

1. Cursor: implementation and code review.
2. GitHub: branches, PRs, audit history and CI.
3. Vercel: frontend/control plane and preview deployments.
4. Managed Postgres/PostGIS: transactional data and spatial policy.
5. Redis: cache/rate limits/short-lived coordination.
6. Kafka-compatible event bus or managed queue: durable ingestion and event processing.
7. Stripe: primary payment rail.
8. Provider adapters: GoCardless, Nuvemshop, Shopify, Glovo/3PL and POS.

## Cost posture

Start with the lowest professional tiers that support the team, then move to enterprise controls when operational requirements justify them. Current Vercel published pricing shows Hobby $0, Pro $20/month and Enterprise custom; Enterprise includes advanced access/security and platform SLAs. Current Cursor published pricing shows Pro $20/month, Pro+ $60/month, Ultra $200/month and Teams Standard $40/user/month.

The infrastructure budget must be driven by database, event, observability, payment and traffic requirements rather than AI-builder price alone.
