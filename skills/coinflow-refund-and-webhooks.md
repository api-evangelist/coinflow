---
name: Refund a payment and reconcile via webhooks
description: Issue a refund against a Coinflow payment, track refund status, and reconcile using signed webhook events.
api: openapi/coinflow-openapi-original.json
operations: [refund-payment, get-refunds, get-refund-via-payment-id, retry-refund, resend-webhook]
---

# Refund a payment and reconcile via webhooks

All operationIds below exist verbatim in `openapi/coinflow-openapi-original.json`.

## Auth
- Merchant-scoped: send the merchant API key in `Authorization`.

## Steps
1. `refund-payment` — PUT `/merchant/payments/{paymentId}/refund` to issue the refund.
2. `get-refund-via-payment-id` — GET `/refunds/payment/{paymentId}` (or `get-refunds`) to check status.
3. `retry-refund` — POST `/refunds/{refundId}/retry` if a refund failed.

## Reconcile via webhooks
- Refund lifecycle emits `Refund`, `Refund Complete`, `Refund Failure`, `Refund Returned`.
- Verify every webhook: extract `t` and `v1` from the `Coinflow-Signature` header, compute HMAC-SHA256 over `"{t}.{raw body}"` with your Webhook Validation Key, and timing-safe compare against `v1`. Verify against the RAW body string, not re-serialized JSON.
- Re-deliver a missed event with `resend-webhook` (POST `/merchant/webhooks/{webhookLogId}`).

## Status semantics
- Transaction states include `REFUNDED`, `CHARGEBACK`, `CHARGEBACK_WON`, `CHARGEBACK_LOST` — see `errors/coinflow-decline-codes.yml`.

## Testing
- Sandbox base URL `https://api-sandbox.coinflow.cash/api`; see `sandbox/coinflow-sandbox.yml`.
