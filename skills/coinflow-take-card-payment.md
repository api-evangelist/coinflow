---
name: Take a card payment with Coinflow
description: Authenticate an end user, tokenize their card, quote fees, and run a checkout charge against the Coinflow sandbox or production API.
api: openapi/coinflow-openapi-original.json
operations: [get-message, get-session-key, verify-token, tokenize, get-fees, card-checkout]
---

# Take a card payment with Coinflow

Use this skill to charge a customer's card. All operationIds are real and exist in `openapi/coinflow-openapi-original.json`.

## Auth
- Server calls send the merchant API key in the `Authorization` header.
- End-user context is established with a session key: call `get-session-key` (sends `x-coinflow-auth-user-id`) and pass the returned key as `x-coinflow-auth-session-key` on subsequent calls. Session keys last 24 hours.
- For wallet-based users, `get-message` returns a message to sign; submit the signature via `x-coinflow-auth-signed-message` and confirm with `verify-token`.

## Steps
1. `get-session-key` — obtain a session key for the end user.
2. `tokenize` — tokenize the card via the pre-built UI SDK (never handle raw PAN yourself; tokenization is PCI-compliant).
3. `get-fees` — POST the cart to quote fees before charging.
4. `card-checkout` — POST `/checkout/card/{merchantId}` to run the charge.

## Idempotency & retries
- Money-movement requests accept a client-generated `idempotencyKey` body field — reuse the same key when retrying so a charge is not double-processed.

## Errors to handle
- `412 Challenge required` — a 3DS/customer challenge is needed; complete it and retry.
- `451 Must complete additional verification` — KYC/compliance step required.
- `423 Customer is Blocked` — do not retry; the customer is blocked.
- `410 Gone` — card validation expired; re-tokenize (revalidate CVV) before reusing card-on-file.
- Card-network declines surface as processing codes — see `errors/coinflow-decline-codes.yml`.

## Testing
- Sandbox base URL `https://api-sandbox.coinflow.cash/api`; test cards + 3DS triggers in `sandbox/coinflow-sandbox.yml` (e.g. Visa `4242 4242 4242 4242`, zipcode `99999` forces a decline).
