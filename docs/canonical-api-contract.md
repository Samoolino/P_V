# Canonical Allocation API Contract

All endpoints are tenant-scoped and require authenticated actor context. Idempotency is mandatory for all mutating financial commands.

## Plan

`POST /v1/plans`

Creates a draft plan.

`POST /v1/plans/{planId}/validate`

Runs deterministic policy and configuration validation.

`POST /v1/plans/{planId}/activate`

Activates only a `POLICY_VERIFIED` and `FUNDING_READY` plan.

## Allocation

`POST /v1/plans/{planId}/allocations`

Request concepts:

```json
{
  "beneficiary_id": "ben_opaque",
  "amount": 50000,
  "currency": "NGN",
  "strategy": "PURPOSE_LOCKED",
  "purpose": "GROCERIES",
  "vendor_ids": ["vnd_example"],
  "store_ids": ["store_example"],
  "product_categories": ["GROCERY"],
  "geo_policy_id": "geo_ikeja",
  "expires_at": "2026-12-31T23:59:59Z"
}
```

## Checkout authorization

`POST /v1/checkouts/authorize`

```json
{
  "plan_id": "pln_123",
  "beneficiary_id": "ben_123",
  "cart_id": "cart_123",
  "store_id": "store_123",
  "delivery_mode": "STORE_PICKUP",
  "idempotency_key": "idem_123"
}
```

Response contains a deterministic decision:

```json
{
  "decision": "APPROVED",
  "authorization_id": "auth_123",
  "allocation_reservations": [
    {"allocation_id": "all_123", "amount": 12000}
  ],
  "policy_version": "policy_17"
}
```

Rejected decisions must include machine-readable reason codes such as `PLAN_INACTIVE`, `ALLOCATION_EXPIRED`, `VENDOR_NOT_ALLOWED`, `STORE_NOT_ALLOWED`, `PRODUCT_NOT_ALLOWED`, `GEO_NOT_ALLOWED`, `KYC_REQUIRED`, `INSUFFICIENT_AVAILABLE_ALLOCATION`.

## Payment

`POST /v1/payments`

Creates the platform's canonical payment record and provider intent reference. The provider adapter owns provider-specific parameters.

`POST /v1/webhooks/{provider}`

Accepts signed provider events. Events are persisted before downstream processing. Duplicate delivery is a no-op after idempotency evaluation.

## Orders

`POST /v1/orders`

Creates a parent order and child vendor orders from an approved cart.

`POST /v1/orders/{orderId}/validate`

Emits `order.validated` only after payment/allocation policy requirements are satisfied.

## Reconciliation

`POST /v1/reconciliation/runs`

Starts a provider reconciliation job.

`GET /v1/reconciliation/runs/{runId}`

Returns matched, missing, duplicate and mismatch counts.

## Security requirements

- OAuth2/OIDC authentication
- RBAC + ABAC
- tenant isolation
- request schema validation
- signed webhook verification
- idempotency keys
- rate limiting
- audit correlation IDs
- no secrets in client payloads
- no NIN/BVN in QR or public UUIDs
