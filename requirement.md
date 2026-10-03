# Grocery Price Compare — Product Plan

Oct 2, 2026 · @Muhammad Anees

## Summary

Build a nationwide basket price comparer for Pakistan, using only prices published online. Start with 300 packaged staple products across every chain that sells online.

- **Scope:** all of Pakistan. Users pick their city and see only the stores that deliver there.
- **Why nationwide works:** branded packaged goods carry a printed retail price set nationally, so prices differ by store, not by city. City changes which stores are available, not the price.
- **No offline data:** no field checks, receipts or shelf scans. Every price is an online price with a timestamp.
- **The gap:** existing Pakistani comparers focus on electronics. Nobody compares a weekly grocery basket across Carrefour, Al-Fatah, Naheed, Metro and Chase Up.
- **The proof:** Fustog does this in Saudi Arabia and holds a 4.7 App Store rating.
- **The money:** affiliate commission on orders, brand placements, and later a B2B price-data feed.

**Verdict:** worth building as a side project. Validate demand in 8 weeks before adding more stores.

## Problem and target users

The same basket costs noticeably different amounts across chains. A Picodi basket study of 17 staples found Carrefour cheapest and Naheed most expensive among five online supermarkets. Shoppers can't check this without visiting each store or app.

**Primary user:** urban middle-class household that orders groceries online managing a monthly grocery budget. Buys from 2–3 chains. Price-sensitive due to inflation.

**Secondary users:**

- Bulk buyers: hostels, small offices, home-based food businesses.
- Deal hunters tracking weekly promotions.
- FMCG brands and analysts wanting competitor shelf prices (B2B, later).

**Core jobs to be done:**

1. Where is my monthly list cheapest today?
2. Is this "sale" actually a discount?
3. Tell me when cooking oil or atta drops in price.

## Competitor landscape

No direct competitor found for supermarket basket comparison in Pakistan. Threats come from adjacent players, not lookalikes.

| Player | Type | What it does | Threat |
|---|---|---|---|
| LowPrice.pk | Price comparer | Compares Daraz, Telemart, iShopping and 340+ stores. Price history and alerts. Includes groceries. | High — closest overlap |
| PriceOye | Price comparer | Compares prices across major online stores. Phones and gadgets focus. | Low |
| PriceMeter.pk | Price comparer | 1,200+ online stores. Shopping lists with offline price estimates. | Medium |
| Durust Daam | Government app | Official rates for produce, meat, poultry in Islamabad. | Low — fresh items only |
| foodpanda pandamart | Quick commerce | Sells groceries; also hosts a Carrefour shop with 10,000+ products. | Medium — owns the order |
| inDrive.Groceries / Krave Mart | Quick commerce | Dark stores, 7,500+ items, 20–30 min delivery. Acquired by inDrive in 2026. | Medium — could add comparison |
| GrocerApp | Online grocer | App-first grocery delivery. | Low |

**Global analogues to study:**

- Fustog (Saudi Arabia) — compares Panda, Carrefour, Lulu and others. Includes weekly promotions. 4.7 rating, 434 ratings.
- HyperDeal (Kuwait) — cross-store deals and shareable shopping lists.
- Awfarli (Lebanon) — real-time comparison plus list-based cheapest store.

**Our wedge:** basket math, not single-product search. "Your 40-item list costs Rs X at Al-Fatah, Rs Y at Carrefour." LowPrice.pk answers "cheapest phone"; we answer "cheapest month of groceries."

## Stores to cover

Cover every chain with an online price catalog, and record which cities each one delivers to.

| Store | Online channel | Main delivery cities | Notes |
|---|---|---|---|
| Carrefour | carrefour.pk web store and app | Karachi, Lahore, Islamabad and others | Biggest catalog; check whether prices change by city |
| Al-Fatah | alfatah.pk web store | Lahore, Islamabad | Catalog is live online |
| Naheed | naheed.pk | Karachi, ships nationwide | Groceries plus electronics |
| Metro | metro-online.pk | Several cities | Wholesale-leaning prices |
| Chase Up | Own online store | Karachi, ships nationwide | Budget segment |
| GrocerApp | Web and app | Lahore and others | App-first grocer |
| Daraz (grocery section) | Web and app | Nationwide | Packaged goods only |
| pandamart, inDrive.Groceries | Apps only | Major cities | Added in v3, where their terms allow |
| Imtiaz, Jalal Sons, Save Mart | No usable online catalog found | Various | Possible direct-feed partners later |

- **Pricing model:** one national price per product per store. If a store turns out to price by city, read it once per city for that store only.
- **City model:** each store has a list of the cities it delivers to. The user's city filters which stores appear in a comparison.
- **Price label:** every price shows "online price, updated <time>". In-store shelf prices may differ, and the app says so plainly.

**Before building:** for Carrefour and Al-Fatah, compare 10 products with a Lahore address and then an Islamabad address. Also confirm each store's delivery cities.

## Daily price collection

Prices come only from online sources, added in three phases. Nobody visits stores and nothing comes from users.

**MVP: store web catalogs.**

- Read each chain's public web catalog: Carrefour, Al-Fatah, Naheed, Metro, Chase Up, GrocerApp and Daraz grocery.
- Read one national price per store, or one per city only for stores proven to price by city.
- Refresh the 300 basket items several times a day and the full catalog daily.
- Keep every price as history and never overwrite it.
- Read only public pages, at a polite rate, following each site's terms.

**v3: app-only stores.** pandamart and inDrive.Groceries, only where their terms allow it, or through a partnership.

**Later: direct feeds.** Offer chains without online catalogs (Imtiaz, Jalal Sons, Save Mart) a free product feed in exchange for traffic and referrals.

**Health rule:** alert when a source returns 0 items, a price jumps more than 40%, or a catalog shrinks by over 20% overnight.

## Product matching

Matching is harder than scraping. "Dalda Cooking Oil 5L Bottle" at one store is "DALDA OIL 5 LTR" at another. Wrong matches destroy trust.

Approach, in order of reliability:

1. **Barcode (EAN/GTIN).** An exact match. Some store sites include it in their product data; check each source's API response.
2. **Normalized attributes.** Parse brand, product, size, unit and pack count into one canonical form, for example `dalda|cooking-oil|5|L|1`.
3. **Fuzzy text plus image similarity.** Score the remaining candidates on name similarity, embeddings and product-photo matching. Auto-accept only above a high threshold.
4. **Human review queue.** Everything in the grey zone goes to an admin screen. Expect to review about 50 a day early on.

Rules:

- Never compare different sizes as equal. Show price per kg or per litre instead.
- Keep a master catalog of canonical products. Store listings map to it many-to-one.
- Start with 300 hand-verified basket items. Expand only after matching accuracy stays above 98%.

## Features

Ship the basket comparison first. Everything else waits until people use it weekly.

| Phase | Feature | Why it matters |
|---|---|---|
| MVP | Basket builder: add items, see the total per store | The core wedge |
| MVP | Cheapest-store split, e.g. "buy these 12 at Carrefour, these 8 at Naheed" | Real savings, and easy to share |
| MVP | Delivery fee and minimum order included in each basket total | The true cost of ordering online |
| MVP | Product search with price per kg or litre | Fair comparison across sizes |
| MVP | Price history chart for each product | Exposes fake sales |
| MVP | Urdu and English, plus WhatsApp sharing of lists | Local fit and free distribution |
| v2 | Price-drop alerts on saved items | Drives retention |
| v2 | Weekly deals feed across stores | A reason to open the app every week |
| v2 | "Order on store" deep links | Affiliate revenue |
| v3 | Add quick-commerce apps (pandamart, inDrive) | Wider coverage |
| v3 | City-by-city grocery inflation index on a public page | PR, SEO and press coverage |
| v3 | Direct feeds from chains without online catalogs | Growth |

**Skip for now:** in-store prices, receipt scanning, our own delivery and checkout, and fresh produce. Fresh prices vary daily and by quality.

## MVP deliverables

The MVP is done when a user in any city can build a basket and see which online store is cheapest today. Each deliverable below is written as a testable outcome.

| # | Deliverable | Done when |
|---|---|---|
| 1 | Daily online price collection | Prices from at least 5 online chains refresh at least once a day; every price is stored with its time, and its city where a store prices by city |
| 2 | Master catalog of 300 staples | 300 basket items are each mapped to their listing in every store that sells them; no unverified match is shown |
| 3 | Unit price normalization | Every product shows price per kg, litre or piece |
| 4 | Product search | A user finds any of the 300 items by name in Urdu or English |
| 5 | Basket builder | A user adds items and quantities and saves the basket without signing up |
| 6 | Store comparison | The basket shows its total at each store, including delivery fee and minimum order, with missing items flagged |
| 7 | Cheapest split | The app suggests the cheapest one-store option and the cheapest two-store split |
| 8 | Price history | Each product shows a price line since tracking began |
| 9 | Freshness label | Every price shows "online price, updated <time>"; anything older than 24 hours is marked stale |
| 10 | Share to WhatsApp | A basket and its comparison can be shared as a link or image |
| 11 | Admin match review | An admin can approve or reject uncertain product matches |
| 12 | Source health alerts | The admin is alerted when a store returns no items or prices jump more than 40% |
| 13 | City selection | A user picks a city once; only stores delivering there appear in comparisons, and the choice is remembered |

**Out of the MVP:** price-drop alerts, deals feed, affiliate links, app-only stores, user accounts beyond saved baskets, fresh produce.

**Success signal after launch:** 300 waitlist signups before launch, then 25% of users still building baskets in week 4. These are suggested targets.

## Monetization

No paywall at launch. Revenue starts once weekly users exist.

| Stream | When | How |
|---|---|---|
| Affiliate on orders | Month 3+ | Deep-link to store app or foodpanda/Daraz shop; commission per order. Needs partner deals. |
| Sponsored placements | Month 4+ | FMCG brands pay to highlight a product or deal. Clearly labelled. |
| Premium tier | Month 6+ | Unlimited alerts, spend insights, family-shared lists. Small monthly fee. |
| B2B price data | Month 9+ | Daily shelf-price feeds and dashboards for FMCG brands, distributors, analysts. Highest margin. |
| Display ads | Fallback | Low CPMs in Pakistan. Use only if others stall. |

**Strongest long-term bet:** B2B data. Price history across chains is valuable to brands. It also survives if consumer growth is slow.

## Risks and legal

Scraping is the biggest legal and operational risk. Get a lawyer's view before monetizing. This is not legal advice.

| Risk | Impact | Mitigation |
|---|---|---|
| A store blocks automated reading or sends a takedown | Lose a source | Polite request rates, public pages only, no login areas; move to direct feeds |
| Terms of service forbid scraping | Legal exposure | Read each site's terms first; prefer stores without explicit bans; seek feeds early |
| Store coverage differs by city | Thin comparison | Show how many stores deliver to the user's city; prioritise chains with national delivery |
| Online price differs from shelf price | User confusion | Label every price "online price" with a timestamp; never claim it is the shelf price |
| Wrong product matches | Trust loss | Barcode first, then human review, and show price per unit |
| Store changes its site layout | Source breaks | Prefer JSON APIs, use health checks, add a fallback parser per store |
| LowPrice.pk adds a basket feature | Competition | Move fast on basket UX, Urdu and grocery depth |
