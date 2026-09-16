# Institutional CSR Allocation Engine v2

## Product constitution

P_V is a subscription-based, multi-tenant, multi-vendor allocation marketplace. Payment processing is infrastructure; the Allocation/Plan Management Engine is the product authority.

A plan owner funds a plan and defines who may receive access, how value is allocated, what purpose it serves, which vendors/stores/products/geographies are permitted, which fulfillment options apply, and how every action is reported and reconciled.

## Non-negotiable invariants

1. Allocation is a quantified, purpose-defined spending entitlement; it is not an unrestricted cash wallet.
2. Beneficiary balances are non-transferable and non-cashable unless a separately approved product policy explicitly permits otherwise.
3. Allocation policy is evaluated before cart authorization and again at checkout.
4. Stripe is the primary payment rail and provider adapters must never become the system of record.
5. The internal ledger is immutable; corrections use compensating entries.
6. Every store has its own transaction identity even when owned by the same vendor organization.
7. NIN/BVN and other KYC identifiers are never exposed as QR/UUID secrets; verification produces an internal identity assertion.
8. Geo evidence is policy input, not proof of identity by itself.
9. Provider webhooks are untrusted input until signature, timestamp, idempotency and schema validation succeed.
10. No production financial authorization is enabled until integration, reconciliation, security and recovery gates pass.

## Logical architecture

```text
Plan Owner
  -> Plan Management
  -> Allocation Policy Engine
  -> Beneficiary Access (QR / UUID / verified identity)
  -> Allocation Wallet / Entitlement
  -> Catalog + Cart
  -> Checkout Policy Guard
  -> Payment Orchestrator
       -> Stripe
       -> GoCardless
       -> future rails
  -> Immutable Ledger
  -> Settlement / Payout
  -> Reconciliation / CDC Analytics

Commerce adapters:
  Nuvemshop / Shopify / ERP / POS / CSV / API

Fulfillment adapters:
  Glovo / other 3PL / internal fleet

Enterprise ingestion:
  SAP IDoc / NetSuite / vendor APIs
       -> Anti-Corruption Layer
       -> Canonical Catalog Events
       -> event bus
       -> read models
```

## Tenant model

`tenant -> organization -> users/roles -> plans -> vendors -> stores -> products -> integrations`.

A vendor organization may own multiple stores. Each store has independent `store_id`, geo identity, terminal identity, inventory state and transaction state. An online store may be affiliated with a parent organization/store without collapsing transaction identity.

## Plan lifecycle

`DRAFT -> CONFIGURED -> VALIDATING -> POLICY_VERIFIED -> FUNDING_READY -> ACTIVE -> COMPLETED`.

Failure/suspension states: `VALIDATION_FAILED`, `FUNDING_FAILED`, `POLICY_CONFLICT`, `SUSPENDED`, `EXPIRED`, `CANCELLED`.

## Allocation policy

Supported first-class strategies:

- fixed amount per beneficiary
- equal distribution
- percentage distribution
- tiered distribution
- scheduled distribution
- milestone distribution
- purpose/category lock
- vendor/store lock
- product/SKU lock
- geography/geofence lock
- combined vendor + product + geography lock

An allocation stores available, reserved, spent, refunded and expired amounts plus policy references.

## Identity/access

Access modes:

- signed QR reference
- random UUID access token
- sequential allocation reference where operationally required
- verified identity assertion after configured KYC tier

Secrets are opaque and revocable. QR payloads contain references, not balances, KYC values or unrestricted financial authority.

## Geo policy

PostGIS is the spatial policy engine. Use a versioned policy model with polygon/multipolygon zones, store points, permitted distance/tolerance, freshness requirements and an explicit fallback behavior. GPS is treated as telemetry and should not be described as a cryptographic proof of physical presence.

Nested policy evaluation:

`global -> country -> region -> city/LGA -> plan zone -> store radius -> branch`.

Catalog visibility may be filtered by policy, but checkout always re-evaluates the authoritative policy server-side.

## Brand/store hierarchy

A plan may authorize:

- a specific store
- a store cluster
- a vendor organization
- a brand within a defined geography

Brand-wide redemption is valid only where every participating branch is explicitly verified and mapped to the authorized vendor hierarchy.

## Checkout authorization

Before capture/settlement the policy engine evaluates:

`recipient + plan + allocation + status + amount + vendor + store + SKU/category + geography + fulfillment + expiry + KYC + idempotency`.

A failed policy evaluation produces a deterministic rejection reason. No client-side flag can override a server-side decision.

## Multi-vendor cart

A parent order may contain child vendor orders. Each child order retains vendor/store/product identity. Allocation reservations are created before payment and atomically committed/released according to the transaction state machine.

With 3PL enabled, cross-vendor fulfillment may use a parent delivery job and child fulfillment records. Settlement remains attributable to each party.

## Payment and ledger

```text
Funding -> Stripe PaymentIntent -> verified provider event -> clearing
       -> allocation ledger -> entitlement

Checkout -> policy authorization -> payment -> ledger entries
        -> vendor settlement -> payout -> reconciliation
```

Ledger invariant: total debits equal total credits for every posted transaction. Provider IDs are references, never ledger truth.

## ERP Anti-Corruption Layer

Inbound SAP/NetSuite/vendor payloads are translated into canonical `CatalogProduct`, `Store`, `InventoryPosition`, `Price` and `CatalogUpdated` events. Raw source payloads are retained for audit, while business logic operates on canonical models.

Do not write directly to a retailer's ERP. ERP access must be least-privilege, scoped, encrypted and independently revocable.

## Event model

Core events:

`plan.created`, `plan.verified`, `plan.activated`, `funding.received`, `beneficiary.verified`, `allocation.created`, `allocation.reserved`, `allocation.consumed`, `allocation.released`, `allocation.refunded`, `checkout.approved`, `checkout.rejected`, `payment.created`, `payment.authorized`, `payment.captured`, `payment.failed`, `payment.refunded`, `order.validated`, `delivery.requested`, `delivery.updated`, `settlement.created`, `settlement.completed`, `payout.completed`, `reconciliation.mismatch`.

Every event carries `event_id`, `event_version`, `trace_id`, `tenant_id`, `occurred_at`, actor/source and correlation references.

## Audit and reporting

Every financial and policy transition is append-only and queryable by sponsor, campaign, plan, beneficiary pseudonymous ID, vendor, store, SKU/category, location policy and delivery reference. PII is separated from analytics identifiers and minimized.

## Security boundary

Frontend -> API gateway -> auth/RBAC/ABAC -> policy services -> financial services -> provider adapters.

Payment secrets, ERP credentials and signing keys remain server-side. PCI-sensitive card handling stays with Stripe-hosted/Stripe-controlled payment surfaces wherever possible.

## Scale target

The 1,000,000-vendor target is an architectural target, not a launch requirement. Initial production should be horizontally scalable with tenant isolation, queue-backed ingestion, indexed PostGIS queries, partitionable ledger/audit tables, idempotent consumers and observable provider adapters.
