---
name: Quote and execute a merchant payout
description: Check payout balance, quote a delegated payout, execute the withdrawal, and poll its status with the Coinflow API.
api: openapi/coinflow-openapi-original.json
operations: [get-payout-balance, get-delegated-payout-quote, do-payout, payout-from-delegated-settlement-wallet, get-withdrawal]
---

# Quote and execute a merchant payout

All operationIds below exist verbatim in `openapi/coinflow-openapi-original.json`.

## Auth
- Payout operations are merchant-scoped: send the merchant API key in `Authorization`.

## Steps
1. `get-payout-balance` — GET `/merchant/withdraws/payout/balance` to confirm available funds.
2. `get-delegated-payout-quote` — POST `/merchant/withdraws/payout/delegated/quote` to quote fees and limits.
3. `do-payout` — POST `/merchant/withdraws/payout` (or `payout-from-delegated-settlement-wallet` for delegated settlement) to execute.
4. `get-withdrawal` — GET `/merchant/withdraws/{withdrawalId}` to poll status.

## Idempotency
- Pass a client-generated `idempotencyKey` in the payout request body and reuse it on retry so a payout is never sent twice.

## Webhooks
- Payout lifecycle emits `Withdraw Pending`, `Withdraw Success`, `Withdraw Failure`, plus rail-specific failures (`Wire Failure`, `Iban Failure`, etc.). See `asyncapi/coinflow-webhooks.yml`.

## Errors to handle
- `402 Payment Required` — payout method unavailable.
- `503 Service Unavailable` — transaction can't be confirmed; fetch a new transaction, re-sign, and resend.

## Testing
- Sandbox payout testing: https://docs.coinflow.cash/guides/payouts/testing-sandbox/testing-withdraws
