---
name: Create a customer and subscribe them to a plan
description: Create a Coinflow customer, list a merchant's subscription plans, and enroll the customer in a plan with a card.
api: openapi/coinflow-openapi-original.json
operations: [create-customer, get-available-plans, create-subscription-card]
---

# Create a customer and subscribe them to a plan

All operationIds below exist verbatim in `openapi/coinflow-openapi-original.json`.

## Auth
- Send the merchant API key in `Authorization`; establish end-user context with a session key (`x-coinflow-auth-session-key`) as in the card-payment skill.

## Steps
1. `create-customer` — POST `/customer` to create the end-user customer record.
2. `get-available-plans` — GET `/subscription/{merchantId}/plans` to list the merchant's plans.
3. `create-subscription-card` — POST `/subscription/{merchantId}/subscribers/card` to enroll the customer in the chosen plan with a tokenized card.

## Lifecycle & webhooks
- Subscription state changes emit webhooks: `Subscription Created`, `Subscription Canceled`, `Subscription Expired`, `Subscription Failure`, `Subscription Concluded` (see `asyncapi/coinflow-webhooks.yml`). Verify the `Coinflow-Signature` (HMAC-SHA256) on every delivery.
- Cancel with `cancel-customer-subscription`; list a customer's subscriptions with `get-customer-subscriptions`.

## Errors to handle
- `422 Unprocessable Entity` — a required field (e.g. address) is missing.
- `451` / `428` — additional verification or re-verification required before the charge can proceed.

## Testing
- Use the sandbox base URL and test cards in `sandbox/coinflow-sandbox.yml`.
