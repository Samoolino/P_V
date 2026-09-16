# ỌJà WA — Institutional Multi-Vendor Allocation Marketplace

ỌJà WA is being structured as a subscription-based, multi-tenant marketplace with a policy-controlled CSR/benefit allocation engine.

## Product authority

The **Plan / Allocation Management Engine** is the business authority. Stripe is the primary payment rail; the internal ledger, policy engine and audit trail remain platform-controlled.

A plan owner can fund a plan, define beneficiaries and allocation strategy, lock spending by purpose/vendor/store/product/geography, select fulfillment rules, brand the customer experience and receive end-to-end reporting.

## Institutional architecture

```text
Plan Owner
  -> Plan Builder
  -> Allocation Policy Engine
  -> Beneficiary Access (QR / UUID / KYC assertion)
  -> Allocation Wallet / Entitlement
  -> Catalog / Cart
  -> Checkout Policy Guard
  -> Stripe Payment Rail
  -> Immutable Ledger
  -> Settlement / Payout
  -> Reconciliation / Audit

ERP / Commerce / POS -> Anti-Corruption Layer -> Canonical Events
3PL / Glovo -> Logistics Router -> Fulfillment Events
```

## Production principles

- allocation is restricted entitlement, not unrestricted cash
- fail-closed checkout policy
- immutable ledger with compensating corrections
- provider-neutral adapters
- store-level transaction identity
- least-privilege enterprise integrations
- idempotent webhooks and commands
- tenant isolation and ABAC/RBAC
- auditability of every financial and policy transition
- production enablement only after testing, reconciliation and recovery gates

## Documentation

- `docs/institutional-architecture-v2.md` — system architecture and invariants
- `docs/production-sequence.md` — implementation sequence and production gates
- `docs/canonical-api-contract.md` — API and checkout contract
- `docs/ai-builder-and-hosting-strategy.md` — engineering tool and hosting decision

## Current branch

`feat/institutional-csr-allocation-v2`

This branch establishes the institutional architecture baseline. Runtime implementation proceeds through the documented production gates; no claim of live payment, ERP or 3PL certification is made until those gates are evidenced.
