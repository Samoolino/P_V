# Production Sequence and Assurance Gates

## Stage 0 — Architecture freeze

Deliverables:
- canonical domain model
- tenant/security model
- plan/allocation state machines
- payment/ledger state machines
- provider adapter contract
- event contract
- OpenAPI contract
- threat model

Exit: no unresolved domain ambiguity on financial authority.

## Stage 1 — Foundation

Build:
- TypeScript service contracts
- PostgreSQL + PostGIS
- Redis for short-lived cache/locks only
- event bus abstraction
- object storage for controlled evidence/assets
- OIDC authentication
- RBAC + ABAC policy layer
- audit/event envelope

Exit tests: migrations, tenant isolation, authorization and idempotency.

## Stage 2 — Plan and Allocation Engine

Build the five-minute plan builder:
1. objective
2. beneficiaries
3. funding
4. allocation strategy
5. restrictions
6. fulfillment
7. branding
8. verification
9. activation

Implement fixed, equal, percentage, tiered, scheduled and locked allocations.

Exit: deterministic policy evaluation and full audit trail.

## Stage 3 — Beneficiary access

Implement:
- opaque UUID access
- signed/revocable QR
- KYC tier abstraction
- beneficiary pseudonymous identifiers
- one-access-per-user-per-plan rule
- allocation wallet/entitlement view

Exit: replay, tampering, duplicate enrollment and authorization tests pass.

## Stage 4 — Vendor and catalog core

Implement:
- vendor organization
- multi-store model
- store geo identity
- products/SKUs/categories
- store-level inventory
- vendor/store verification
- online-store affiliation
- API/CSV ingestion

Exit: vendor/store/product policy tests pass.

## Stage 5 — Checkout policy engine

Evaluate before cart authorization and again at final checkout:
- plan status
- beneficiary
- allocation availability
- vendor/store
- product/category
- geography
- expiry
- KYC tier
- delivery policy
- idempotency

Exit: all negative cases fail closed.

## Stage 6 — Stripe payment core

Implement provider adapter around Stripe PaymentIntents/Checkout as appropriate. Keep provider IDs as external references. Use verified webhooks, idempotency keys and explicit payment state transitions.

Exit: test-mode funding -> payment event -> ledger -> allocation -> refund -> reconciliation.

## Stage 7 — Ledger, settlement and reconciliation

Implement immutable double-entry ledger, reservations, releases, vendor payable, settlement, payout and daily provider reconciliation.

Exit invariants:
- debits = credits
- no double spend
- no negative available allocation
- every payout maps to posted payable
- provider totals reconcile to internal records.

## Stage 8 — Enterprise commerce ingestion

Add Anti-Corruption Layer adapters for SAP IDoc, NetSuite/vendor API and canonical JSON events. Do not write to retailer ERP databases. Add schema validation, dead-letter queues, replay and source-payload retention with access controls.

Exit: deterministic catalog/inventory mapping under replay and out-of-order events.

## Stage 9 — Nuvemshop / Shopify / POS

Connect commerce and POS adapters to canonical catalog/order/checkout contracts. POS is a channel, not a ledger.

Exit: sandbox order and inventory synchronization plus policy-controlled checkout.

## Stage 10 — 3PL / Glovo

Implement provider-neutral logistics router:
- primary route
- fallback route
- dispatch request
- webhook telemetry
- delivery status normalization
- cancellation/retry
- delivery-to-order reconciliation

Exit: simulated cross-vendor parent order with child fulfillments.

## Stage 11 — Observability and resilience

Implement:
- distributed trace IDs
- metrics and SLOs
- structured logs
- alerting
- queue lag monitoring
- provider health
- backup/restore
- incident runbooks
- disaster recovery drill

Target RPO/RTO should be tested rather than assumed.

## Stage 12 — Institutional certification

Evidence pack:
- architecture decision records
- OpenAPI
- threat model
- dependency/SBOM report
- automated test report
- load test report
- reconciliation report
- backup/restore evidence
- access review
- provider sandbox evidence
- production canary evidence
- operational sign-off

Production status is granted only after all mandatory gates pass.

## SLA interpretation

The supplied 200ms ERP ingest, 35,000 concurrent requests, >=85% coverage, RPO <=30s and RTO <=5m are treated as engineering acceptance targets. They should be validated with defined test methodology, workload shape, percentile definitions and exception handling before becoming contractual SLAs.
