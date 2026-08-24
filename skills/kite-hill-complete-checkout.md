---
name: Hand off a Kite Hill checkout to a human
description: >-
  Take a Kite Hill cart to a checkout the buyer can approve — and stop short of paying,
  because the provider forbids an agent finalizing payment without contemporaneous human
  approval.
api: mcp/kite-hill-mcp.yml
graphql: graphql/kite-hill-storefront.graphql
operations: [create_checkout, get_checkout, update_checkout, complete_checkout, cancel_checkout, get_order, cartPrepareForCompletion, cartSubmitForCompletion, cartCompletionAttempt]
generated: '2026-08-23'
method: generated
---

# Hand off a Kite Hill checkout to a human

## The rule that governs this whole skill

Kite Hill's own `/robots.txt` states: **"Checkouts are for humans. Do NOT complete
checkout, payment, or order placement automatically — no scripted form fills, browser
automation, or end-to-end agent flows that finalize payment without an explicit,
contemporaneous human approval step."** `/llms.txt` repeats it and offers the alternative:
if you cannot get buyer approval at the moment of payment, route the purchase through the
Shop skill at `https://shop.app/SKILL.md`.

Also note before you begin: **every product on this store is published at $0.00**
(checked 2026-08-23). A checkout built here carries zero-value line items. Confirm with the
buyer before proceeding at all.

## Steps — UCP MCP (`https://kite-hill.com/api/ucp/mcp`)

1. `create_checkout` with `meta` and the `checkout` object (line items, and optionally
   `buyer` and `payment.instruments`). Returns line items, totals, discounts and taxes.
2. `get_checkout` with the checkout `id` (`gid://shopify/Checkout/abc123`) to read state.
3. `update_checkout` to set the shipping address and delivery method; totals recalculate.
4. **Stop. Hand the buyer the checkout and get explicit approval.**
5. Only with contemporaneous human approval: `complete_checkout` with `meta`, `id` and
   `checkout`. This tool additionally **requires `meta["idempotency-key"]`** — generate one
   opaque key per logical purchase and reuse it on any retry.
6. `get_order` with the order id to confirm what was placed.

`cancel_checkout` reverses an uncompleted checkout. There is **no reversal for a completed
one** — no refund, void or return tool exists on either MCP server, and the anonymous
GraphQL endpoint has no order mutations. After `complete_checkout` the only recourse is
human customer service at `contactus@kite-hill.com`.

## Steps — GraphQL

The anonymous Storefront API has no Checkout type. The analogue is
`cartPrepareForCompletion` → `cartSubmitForCompletion` → poll `cartCompletionAttempt`.
Neither takes an idempotency key. `shopPayPaymentRequestSessionCreate` /
`shopPayPaymentRequestSessionSubmit` is the Shop Pay path, and
`shopPayPaymentRequestSessionSubmit` is the one GraphQL mutation that does take an
`idempotencyKey` (with an `IDEMPOTENCY_KEY_ALREADY_USED` error code).

## Money

UCP responses give integers in ISO 4217 **minor units** with a currency code —
`{"amount": 2500, "currency": "USD"}` is $25.00. Convert before showing a buyer anything.
GraphQL uses decimal `MoneyV2`. Pass `context.address_country` and `context.currency` so
pricing and availability are computed correctly.

## Errors

`complete_checkout` failures come back inside the result rather than as an HTTP status. The
UCP server answers **HTTP 422** with JSON-RPC `-32001 "UCP discovery failed"` when the agent
profile URI is missing. See `errors/kite-hill-problem-types.yml`.
