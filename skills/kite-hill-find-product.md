---
name: Find and compare Kite Hill products
description: >-
  Search Kite Hill's plant-based catalog and resolve a specific variant, using either of the
  two anonymous MCP servers or the Storefront GraphQL API — and check the price before you
  quote one, because this catalog publishes $0.00.
api: mcp/kite-hill-mcp.yml
graphql: graphql/kite-hill-storefront.graphql
operations: [search_catalog, lookup_catalog, get_product, get_product_details, search, products, predictiveSearch, product, productByHandle]
generated: '2026-08-23'
method: generated
---

# Find and compare Kite Hill products

Kite Hill makes plant-based dairy from almond milk — almond milk yogurts, Greek-style
yogurts, cream cheese and sour cream alternatives, ricotta, dips, butter alternative, and
filled pastas. Your job is to get from a shopper's intent to one concrete **variant id**.

## Read this first

Every product on this storefront is published at **$0.00**. All 30 products in
`/products.json` return `price "0.00"` with `available: true`, the GraphQL `priceRange`
returns `0.0 USD`, and a live `search_catalog` call returned `price_range.min/max` of `0`
for every product (checked 2026-08-23). **Do not quote a price from this store.** Kite Hill
sells through grocery retailers — Whole Foods, Target, Sprouts, Publix and others — and its
own answer to "where do I buy this" is the store locator at
`https://kite-hill.com/pages/store-locator`.

## Which surface to use

All three are anonymous — no key, no token.

| Need | Surface |
|---|---|
| Natural-language intent ("unsweetened greek yogurt") | MCP `search_catalog` on either server |
| Resolve several product/variant ids at once | MCP `lookup_catalog` at `/api/ucp/mcp` |
| Full product detail with live availability | MCP `get_product` at `/api/ucp/mcp` |
| Exact, structured queries and full field control | GraphQL at `https://kite-hill.com/api/2026-04/graphql.json` |

Two MCP endpoints serve overlapping catalog tools:

- `https://kite-hill.com/api/ucp/mcp` — 13 tools, the full commerce lifecycle. Every tool
  requires `meta["ucp-agent"]["profile"]`, a resolvable agent profile URI.
- `https://kite-hill.com/api/mcp` — 5 tools, catalog/cart/policy only. `get_product_details`
  here takes plain `product_id`, `options`, `country`, `language`, and a `tools/call` of
  `search_catalog` succeeded with no `meta` object at all.

## Steps — MCP path

1. Call `search_catalog` with a natural-language `query`, structured `filters`, or both —
   the tool requires at least one. On the UCP server also pass `meta` with your agent
   profile URI.
2. Results are paginated with a deliberately small first page. Read `pagination.cursor` and
   page forward rather than asking for a huge page.
3. Resolve the exact item with `get_product` (UCP) or `get_product_details` (Storefront).
   **Without `options`, `get_product_details` returns the first available variant**, which
   may not be the flavor or size the shopper asked for.
4. Use `lookup_catalog` when holding several ids — it resolves `gid://shopify/Product/...`
   and `gid://shopify/ProductVariant/...` in one request.

## Steps — GraphQL path

1. `products(first:, query:, sortKey:)` or `search(query:, types: [PRODUCT], productFilters:)`
   to list candidates. Select
   `id handle title productType availableForSale priceRange { minVariantPrice { amount currencyCode } }`.
2. `product(id:)` or `productByHandle(handle:)` for detail. Select
   `variants(first: 25) { nodes { id title sku availableForSale quantityAvailable price { amount currencyCode } selectedOptions { name value } } }`.
3. `predictiveSearch` covers the typeahead case.
4. No MCP tool lists collections. To enumerate `original-yogurt`, `greek-yogurt`,
   `cream-cheese`, `sour-cream`, `dips`, `ricotta-alternative`, `pasta`,
   `butter-alternative` or `whole30`, use GraphQL `collections` / `collectionByHandle` —
   a real gap recorded in `mcp/kite-hill-tool-crosswalk.yml`.

## Rules

- **Never quote a price you did not read from a response** — and on this store, what you
  read is zero. Say the store does not publish a price and point at the store locator.
- Check `availableForSale` / `availability.available` before promising anything.
- UCP tools return money in **ISO 4217 minor units** (`{"amount": 600, "currency": "USD"}`
  is $6.00); GraphQL returns a decimal `MoneyV2`. The two surfaces disagree in format.
- The shop prices in USD and `shipsToCountries` lists 28 countries.
- Sampled variants carry an empty `sku` and the title `Default Title` — there is no
  meaningful variant axis on most items.

## Errors

GraphQL returns HTTP 200 with an `errors[]` array; check it before reading `data`. Some
fields are scope-gated even anonymously — `productTags` returns `ACCESS_DENIED` requiring
`unauthenticated_read_product_tags`. Every response carries `extensions.cost`; the API is
query-cost throttled, not header rate limited. MCP errors arrive as JSON-RPC `error`
objects (the UCP server answers HTTP 422 for a missing agent profile). See
`errors/kite-hill-problem-types.yml`.
