---
name: shopify-store-audit
owner: launifycorp
category: Ecommerce
description: Pre-launch readiness audit of a Shopify store from its URL alone, no admin access. Reads what the storefront publishes about itself — theme, currency, catalogue JSON, and the fixed policy URLs that either resolve or do not — and reports what a first customer would hit. Never completes a checkout, never judges whether a policy is legally adequate.
version: v1
license: MIT
updated: 2026-08-30
recommended: false
security_checked: true
url: https://emdly.com/skills/launifycorp/shopify-store-audit
raw: https://emdly.com/raw/launifycorp/shopify-store-audit.md
install: npx @emdly/cli add launifycorp/shopify-store-audit
---

# Shopify store readiness

Give it a store URL. It checks the things a first customer will hit, from the customer's side of the counter, and reports what would break or embarrass the store on day one.

This is the audit you can run without anyone handing you admin access. That constraint is the point: it sees exactly what a visitor, a crawler and a regulator can see, which is a different and more honest list than what the admin shows you. It also means it cannot see tax rates or shipping zones, and it says so rather than guessing.

Shopify makes this possible because a storefront volunteers a great deal about itself. The theme names itself, the shop object carries the currency and country, the catalogue is readable as JSON, and every store policy lives at a fixed URL that either resolves or does not. Most pre-launch problems are visible from outside if you know where to look.

## When to use

- Before a launch, or before a soft launch, on a store nobody has stress-tested.
- After a migration onto Shopify, when the old site's URLs and the new store's settings have not been reconciled.
- Before opening a new market, to see what a customer in that country actually gets.
- On a store you have inherited and do not yet have admin access to.
- As a recurring check on a live store, where policies and pages drift.

Not for: theme design quality (`launifycorp/design-review`), search indexing (`launifycorp/seo-audit`), the Google Shopping feed (`cartlift/product-feed-optimizer`), or checkout funnel analytics (`shopmetric/ga4-funnel-analyst`).

## Input

**The store URL. Nothing else is required.**

If none was supplied, ask and stop. Do not infer a domain.

Useful when offered, and named as absent when not:

| also useful | what it unlocks |
|---|---|
| the markets and currencies the store intends to sell in | Pass 6 has nothing to compare against |
| the previous site's URLs, if this is a migration | the redirect check in Pass 8 |
| whether the store is pre-launch or live | changes a password page from a finding into a confirmation |

`FETCH` means an HTTP request returning status, headers and body. `BROWSER` means the same URL rendered. Both are used; where only one is available, say which passes did not run.

## What this audit cannot see

Put this in every report. Half of "store readiness" lives in the admin, and this audit is outside it.

- **Tax rates, tax regions, and whether prices include tax.** Not exposed. The storefront shows a number; whether that number is correct for a customer in a given country is an admin setting and an accountant's question.
- **Shipping zones and rates.** Rates are calculated at checkout against a real address and a real cart. You can read the shipping *policy* page; you cannot read the rate table.
- **Which payment providers are actually enabled.** Footer icons are a theme setting and are frequently wrong.
- **Notification email templates**, order routing, inventory locations, staff permissions, app configuration, or anything else behind the admin login.
- **Whether checkout completes.** Testing that means placing an order, which this skill does not do. See Stop 1.

An audit that omits this list reads as a full readiness sign-off, and it is not one.

## Severity

| level | meaning |
|---|---|
| `BLOCKING` | a customer cannot buy, or the store is publishing something it should not be. |
| `LEGAL` | a required policy or disclosure is missing or unreachable. Separate from blocking because it is an obligation, not a malfunction — and because it is the thing pre-launch stores skip. |
| `TRUST` | the customer can buy, but something here makes them hesitate or sets them up for a surprise. |
| `POLISH` | real, small, last. |

`LEGAL` findings are about **presence and reachability only**. Whether a policy's text is adequate for a jurisdiction is not a question this skill answers. See Stop 2.

---

## Pass 1 — Is this Shopify, and is it open

```bash
S="https://store.example.com"
# Shopify identifies itself in the response headers
curl -sSI "$S" | grep -iE 'x-shopid|x-shardid|powered-by|x-sorting-hat'
# a store that is not launched yet parks visitors here
curl -sS -o /dev/null -w '%{http_code} %{url_effective}\n' -L "$S/password"
```

- **Not Shopify.** Say so and stop; every path below is Shopify-specific.
- **Password page.** The store is not public. If the operator called it pre-launch, that is a confirmation and the audit continues on what is reachable. If they called it live, it is `BLOCKING` and the first finding.
- **Theme identity**, from the rendered page:

```js
({
  shop: window.Shopify?.shop,
  currency: window.Shopify?.currency,          // {active, rate}
  country: window.Shopify?.country,
  locale: window.Shopify?.locale,
  theme: {
    name: window.Shopify?.theme?.name,
    role: window.Shopify?.theme?.role,          // "main" is the published theme
    id: window.Shopify?.theme?.theme_store_id
  },
  designMode: window.Shopify?.designMode ?? false
})
```

`theme.role` should be `main`. Anything else means you are looking at a preview, and every finding below belongs to a theme that is not the live one — say so and get the right URL.

## Pass 2 — What the store says about itself

```bash
curl -sS "$S/meta.json" | jq .
```

Record the declared currency, country and locale, and hold them against every price and every claim later in the audit. A store whose `Shopify.currency.active` is `USD` while the footer says "prices in EUR" has one of the two wrong, and the customer finds out at checkout.

## Pass 3 — Is there anything to buy

Shopify exposes the catalogue as JSON without authentication. This is the fastest way to see the shape of the store, and it is also where pre-launch stores leak — see Stop 5.

```bash
# paginated; 250 is the ceiling per page
curl -sS "$S/products.json?limit=250&page=1" \
| jq '{count: (.products | length),
       noImage:      [.products[] | select((.images | length) == 0) | .handle],
       noBody:       [.products[] | select(.body_html == "" or .body_html == null) | .handle],
       zeroPrice:    [.products[] | select(any(.variants[]; (.price | tonumber) == 0)) | .handle],
       noSku:        [.products[] | select(any(.variants[]; .sku == "" or .sku == null)) | .handle],
       unavailable:  [.products[] | select(all(.variants[]; .available == false)) | .handle]}'
```

- **Zero products** on a store presented as ready is `BLOCKING`.
- **A product with no image** converts worse than one that does not exist, and it is visible from the JSON.
- **A price of 0** is either a configuration error or a free item; check which, and say which you concluded.
- **Every variant unavailable** means the product page is a dead end. Count them.
- Also fetch `/collections/all` and check it returns products. A store whose catalogue is fine but whose "all" collection is empty has a navigation problem, not a catalogue problem.

**Pagination and politeness.** Walk pages until a request returns an empty `products` array. Cap your walk, say where you capped, and pace the requests — this is someone's storefront, not an API you have been given a budget on.

## Pass 4 — Policies

**The most checkable readiness item in this audit, and the one most often missed.** Shopify publishes each store policy at a fixed path. If the path 404s, the policy is not set; there is nothing to interpret.

```bash
for p in refund-policy privacy-policy terms-of-service shipping-policy \
         legal-notice subscription-policy contact-information; do
  printf '%-24s ' "$p"
  curl -sS -o /dev/null -w '%{http_code}\n' -L "$S/policies/$p"
done
```

Shopify links these automatically: its documentation states that once added, policies *"are automatically linked in the footer of your checkout pages"*, with the return policy shown on the order review page and the shipping policy on product pages and in the cart. So a missing policy is not only absent from the site — it is a gap in checkout, where the customer is deciding.

Judge:

- **A 404 is `LEGAL`.** Name which policy.
- **A 200 with placeholder text** — Shopify ships templates, and stores publish them unedited. Look for `[COMPANY NAME]`, `[insert`, `lorem`, or a policy that never names the store. That is `LEGAL` too, and it is worse than absent because it looks done.
- **Subscription policy** is only expected if the store actually sells subscriptions. Do not report it missing otherwise.
- **A policy that contradicts the site** — a 14-day return window in the policy and "30-day returns" in the header — is `TRUST`, and it is the kind of thing that becomes a chargeback.

## Pass 5 — The buying path, up to where you must stop

```bash
curl -sS "$S/cart.js" | jq '{item_count, currency, items: (.items | length)}'
```

Walk the path a customer walks: product page, add to cart, view cart, reach the checkout button. **Stop there.** Do not proceed through checkout, do not enter card details, do not place an order. See Stop 1.

What is visible short of that:

- Does add-to-cart work at all, and does `/cart.js` reflect it.
- The currency in `/cart.js` against the currency shown on the product page.
- Whether the checkout button leads to Shopify checkout on the shop's own domain or a `.myshopify.com` one. A checkout on the raw myshopify domain mid-purchase is a trust drop and often means the custom domain was configured for the storefront only.
- Payment method icons in the footer — and report them as **claimed, not verified**, because they are a theme setting and not a live list of what is enabled.
- Whether prices are shown with or without tax, and whether the store says which. In markets where tax-inclusive display is expected, silence is the finding.

## Pass 6 — Markets and localisation

- Locale-prefixed paths (`/en-de/`, `/fr/`) and whether they resolve.
- `hreflang` annotations, and whether each version lists itself and the others.
- The currency switcher, if there is one, and whether the switch survives a page load.
- `Shopify.country` against the market the operator named.

```bash
curl -sS "$S/browsing_context_suggestions.json" | jq .
```

If the operator named no markets, say Pass 6 was run for presence only and could not judge coverage.

**A store that geolocates.** Everything you measured came from one location. Say so, and say that a customer elsewhere may see different currency, different availability, or a redirect you never hit.

## Pass 7 — Where a lost customer lands

- `/search?q=` with a term from the catalogue, and with a term that matches nothing. Does the empty state help or dead-end?
- A URL that does not exist. Is there a real 404 page with a way back, or the theme's default?
- The contact route. Is there a form, an address, a company identifier, and a human-reachable channel? `/policies/contact-information` is where Shopify puts this if the merchant filled it in.
- The footer: does it carry the policy links Shopify generates, or has the theme dropped them?

## Pass 8 — Indexing and duplicate URLs

- `/robots.txt` — Shopify generates a default. Compare it against what is there; a hand-edited robots on a Shopify store is unusual and worth naming.
- `/sitemap.xml` and the child sitemaps it points at. Do the products in the sitemap match the products in `/products.json`?
- **The classic Shopify duplicate.** A product is reachable at `/products/<handle>` and at `/collections/<collection>/products/<handle>`. Check the canonical on the collection-scoped URL points at the bare `/products/` one.
- If this is a migration and the old URLs were supplied, sample them and record which 404 rather than redirect. A missing redirect map is `BLOCKING` on a migration and invisible from anywhere else.

> Thresholds above are defaults; report the thresholds you used.

## Rules

- **Evidence with every finding.** A status code, a JSON field, a quoted line. "Best practice" is not evidence.
- **Order by severity, then by how many customers hit it.** A `TRUST` finding on every product page outranks one on a single page.
- **Say what you could not see**, every time, in the words of the section above. The gap is part of the finding.
- **Never state a tax or legal conclusion.** Report presence, reachability and contradiction. Adequacy is not yours.
- **Distinguish claimed from verified**, especially for payment methods, shipping promises and delivery times. The storefront makes claims; this audit checks that they are consistent, not that they are true.
- **Pace your requests.** This is a live storefront and possibly a small one.
- **A clean pass is reported in one line.** Silence reads as unchecked.

## Output format

```
SHOPIFY READINESS · https://store.example.com
Checked 2026-08-30 · storefront only, no admin access · one location

Shopify confirmed (x-shopid present). Theme "Dawn" role=main. Store is public.
Declared: currency EUR · country DE · locale de

CANNOT SEE: tax rates and regions, shipping zones and rates, which payment
providers are enabled, notification templates, anything in the admin. Checkout
was not completed — no order was placed.

BLOCKING — 1
1. 14 of 62 products have no image.
   /products.json?limit=250 · 62 products across 1 page
   handles: winter-mug, tote-navy, tote-black, … (14 listed in appendix)
   These have live product pages a customer can reach from /collections/all.

LEGAL — 2
2. Three store policies are not set.
   /policies/refund-policy       404
   /policies/shipping-policy     404
   /policies/contact-information 404
   /policies/privacy-policy      200
   /policies/terms-of-service    200
   /policies/legal-notice        404  (not expected in this market, not counted)
   /policies/subscription-policy 404  (store sells no subscriptions, not counted)
   Shopify links these automatically in the checkout footer, and the shipping
   policy is shown on product pages and in the cart, so these are gaps at the
   point of purchase, not only missing pages.
   Presence only. Whether the two that exist are adequate is not assessed.

3. The privacy policy is the unedited Shopify template.
   /policies/privacy-policy contains "[COMPANY NAME]" three times and never
   names the store. Published, and worse than absent, because it looks done.

TRUST — 2
4. Checkout leaves the custom domain.
   Cart button -> https://store-example.myshopify.com/checkouts/...
   The customer types their card on a domain that is not the one they trusted.

5. The product page shows prices without saying whether tax is included.
   Declared country is DE, where a tax-inclusive display is what a shopper
   expects. The storefront makes no statement either way. Whether the setting
   is correct is an admin question this audit cannot reach.

POLISH — 1
6. Collection-scoped product URLs carry the right canonical, but the sitemap
   lists 62 products while /collections/all shows 58. Four are in no collection.

CLEAN
Password page: not present, store is public.
/cart.js works; currency EUR matches the product page.
robots.txt is Shopify's default, unmodified.
404 page is the theme's, with search and a route back to the catalogue.
hreflang: not applicable, single locale.

NOT RUN
Migration redirects: no previous URLs supplied.
Markets coverage: no target markets stated, so Pass 6 checked presence only.
```

The refusal:

```
NOT AUDITED

No store URL supplied.

This audit starts from one storefront URL and works outward from what the store
publishes. Auditing a domain inferred from the conversation produces a confident
report about someone else's shop.

Send the URL. Useful alongside it: which markets and currencies you intend to
sell in, whether the store is pre-launch or live, and — if this is a migration —
a sample of the old site's URLs.
```

## Edge cases

- **No URL.** Ask and stop.
- **Not a Shopify store.** Say so in the first line and stop. Nothing below Pass 1 applies.
- **Password-protected.** Report Pass 1 and stop, unless the operator supplies the storefront password and confirms they own the store. Then continue and say the audit ran behind the password.
- **`/products.json` returns 404 or an empty array on a store that clearly has products.** Some stores disable it, and some themes proxy it. Not a finding on its own; fall back to `/collections/all` and the sitemap, and say the JSON route was unavailable.
- **A very large catalogue.** Sample. Walk the first pages, state how many you walked and how many products the sitemap claims, and audit a sample rather than everything.
- **Bot protection or a challenge page.** What you fetched is the interstitial and every measurement from it is worthless. Stop and say so.
- **A headless storefront** — Hydrogen, or a custom front end on the Storefront API. The `/policies/*` paths and `/products.json` may not exist on the customer-facing domain even though the store is fine. Check the `.myshopify.com` origin as well and say which host each finding came from.
- **A store mid-migration**, where the theme is new and the content is not. Expect placeholder policies and missing images; report them as normal findings and let the operator decide what is expected.
- **A single-product store.** Most of Pass 3 collapses to one row. Say so rather than reporting a thin catalogue as a finding.
- **A policy page that exists but is empty.** A 200 with no content is worse than a 404 and reads the same in a status-code sweep. Check the body length.
- **The store geolocates or redirects by country.** Everything came from one place. Say it, and do not report a missing market you were simply not shown.
- **Prices that differ between `/products.json` and the rendered page.** Usually a currency conversion or a market-specific price list. Report both numbers and do not conclude which is right.

## Stop and hand back

1. **Never complete a checkout and never place an order.** A test order charges a real card or consumes real inventory, fires notification emails, and lands in someone's accounting. Walk to the checkout button and stop. If the operator wants checkout tested end to end, that is theirs to do, with Shopify's own test mode.
2. **Never judge whether a policy's content is legally adequate.** This audit reports that a policy exists, is reachable, is not a template placeholder, and does not contradict the site. Whether it satisfies consumer law in a given market is a lawyer's answer, and a confident wrong one here costs the merchant more than a missing page.
3. **Never state a tax conclusion.** Whether a displayed price is tax-inclusive, whether a rate is right, whether a market needs registration — none of that is visible from the storefront and all of it is expensive to get wrong.
4. **Never enter real personal data, a real address, or a real card** into any form on the store. If a form must be exercised, say what would need to be submitted and let the operator do it.
5. **Anything the store is exposing that it did not mean to.** `/products.json` can reveal products that are published but in no collection — unreleased items, internal SKUs, staging products. Discount codes sometimes sit in the theme's JavaScript. Report the pattern and the count to the store's owner, name where you found it, and do not list the codes or the unreleased products in a report that circulates further.
6. **A store you were not asked to audit.** One storefront, at a normal browsing pace. Do not walk a competitor's whole catalogue through `/products.json`; it is someone's commercial data and the request volume is visible to them.
7. **A finding that requires admin access to confirm.** Say what you observed, say what setting would explain it, and hand it over. Do not ask for admin credentials to close the loop.

## Sources

The policy paths and Shopify's statement that store policies *"are automatically linked in the footer of your checkout pages"* come from Shopify's Help Center page on adding store policies. The public storefront endpoints — `/products.json`, `/collections/<handle>/products.json`, `/cart.js`, `/meta.json`, `/search/suggest.json`, `/browsing_context_suggestions.json` — are undocumented in the developer reference but long-standing and widely relied on; treat them as conventions that can change, and never build a finding on one without a fallback.

## License

MIT
