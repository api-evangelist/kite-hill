---
name: Build a Kite Hill cart
description: >-
  Assemble a cart on Kite Hill's storefront through the UCP or Storefront MCP server, or the
  Storefront GraphQL cart mutations — with no idempotency key available, so treat every
  mutation as at-most-once.
api: mcp/kite-hill-mcp.yml
graphql: graphql/kite-hill-storefront.graphql
operations: [create_cart, get_cart, update_cart, cancel_cart, cartCreate, cartLinesAdd, cartLinesUpdate, cartLinesRemove, cartBuyerIdentityUpdate, cartDeliveryAddressesAdd, cartSelectedDeliveryOptionsUpdate, cartDiscountCodesUpdate, cartNoteUpdate]
generated: '2026-08-23'
method: generated
---

# Build a Kite Hill cart

## Before you start

You need a resolved **variant id** (`gid://shopify/ProductVariant/...`) — see
`kite-hill-find-product.md`. On the UCP server every tool call needs
`meta["ucp-agent"]["profile"]`.

## Steps — UCP MCP

1. `create_cart` with `meta` and a `cart` object carrying the line items.
2. `get_cart` with the cart `id` to read back lines, totals and any discounts.
3. `update_cart` with `meta`, `id` and the changed `cart` fields.
4. `cancel_cart` with `meta` and `id` to abandon it. This is the only published reversal
   for a cart, and **no window is stated** — see the `reversibility` block in
   `conventions/kite-hill-conventions.yml`.

## Steps — Storefront MCP

`update_cart` on `/api/mcp` is one consolidated mutation. Pass `cart_id` plus any of
`add_items`, `update_items`, `remove_line_ids`, `buyer_identity`,
`delivery_addresses_to_add`, `delivery_addresses_to_replace`,
`selected_delivery_options`, `discount_codes`, `gift_card_codes`, `note`. Omit `cart_id`
with `add_items` to create a new cart. `get_cart` returns the cart plus its `checkout url`.

## Steps — GraphQL

`cartCreate` → `cartLinesAdd` / `cartLinesUpdate` / `cartLinesRemove` →
`cartBuyerIdentityUpdate` → `cartDeliveryAddressesAdd` /
`cartSelectedDeliveryOptionsUpdate` → `cartDiscountCodesUpdate` / `cartGiftCardCodesAdd` →
`cartNoteUpdate`. One consolidated MCP `update_cart` collapses all ten.

## Rules

- **There is no idempotency key on any cart operation.** `meta["idempotency-key"]` exists on
  `complete_checkout` and nowhere else. A retried `update_cart` adds the items again, so
  reconcile with `get_cart` instead of retrying blind.
- **There is no dry-run.** Nothing on this surface previews a mutation.
- Cart mutations return typed `CartUserError` lists inside the payload (`field`, `message`,
  `code`) — read those, not just `errors[]`.
- `cartClone`, `cartMetafieldsSet`, `cartMetafieldDelete`, `cartAttributesUpdate` and
  `cartPaymentUpdate` exist in GraphQL and have **no** MCP tool equivalent.
- Do not use `/cart.js`. Kite Hill's `robots.txt` disallows it and tells agents to use
  UCP/MCP instead.
