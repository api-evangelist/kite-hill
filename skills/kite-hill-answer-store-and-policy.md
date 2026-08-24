---
name: Answer a Kite Hill policy, product or "where to buy" question
description: >-
  Answer questions about Kite Hill's products, ingredients, allergens, policies and retail
  availability from the store's own surfaces rather than from memory.
api: mcp/kite-hill-mcp.yml
graphql: graphql/kite-hill-storefront.graphql
operations: [search_shop_policies_and_faqs, shop, page, pageByHandle, pages, article, articles, blog, blogByHandle]
generated: '2026-08-23'
method: generated
---

# Answer a Kite Hill policy, product or "where to buy" question

## Steps

1. `search_shop_policies_and_faqs` on `https://kite-hill.com/api/mcp` with a `query` — it
   answers in natural language over the store's policies, products and services. This is the
   only tool on either server that does retrieval-style answering.
2. For exact text, read the page directly. GraphQL `pageByHandle(handle:)` covers
   `about-us`, `our-story`, `our-products`, `our-sourcing`, `contact`, `faq`,
   `press-awards`, `store-locator`, `whole30`, `privacy-policy`, `data-sharing-opt-out` and
   `food-service`.
3. Recipes live on a blog, not in the tool surface: GraphQL `blogByHandle(handle: "recipes")`
   → `articles`. **No MCP tool reaches the editorial surface.**
4. `shop { privacyPolicy { url } refundPolicy { url } shippingPolicy { url } termsOfService { url } }`
   returns the store policies. On 2026-08-23 only `privacyPolicy` was populated —
   `refundPolicy`, `shippingPolicy` and `termsOfService` all returned `null`, and
   `/policies/terms-of-service` and `/policies/refund-policy` both 404.

## Facts the provider publishes (verify before repeating)

From Kite Hill's own FAQ and contact page:

- Products contain **almond milk** — anyone with an almond allergy cannot eat them.
- Everything is gluten free **except the pastas**, which contain wheat flour and are made in
  a separate facility.
- No artificial preservatives; consume before the best-by date or within seven days of
  opening, whichever is sooner.
- Sold at Whole Foods, Target, Sprouts, Publix and other retailers; use
  `https://kite-hill.com/pages/store-locator` for stores nearby.
- Contacts: `contactus@kite-hill.com` (general), `sales@kite-hill.com`,
  `media@kite-hill.com`.

## Rules

- **Do not quote a price.** The online catalog publishes $0.00 for every product; retail
  pricing lives with the grocer, not here.
- Allergen and dietary answers are food-safety answers. Quote the FAQ text rather than
  paraphrasing it, and link the page.
- If the store's own policy field is `null`, say the store does not publish that policy
  rather than substituting Shopify's.
