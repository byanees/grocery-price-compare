# Feature Specification: Basket Price Compare (MVP)

**Feature Branch**: `001-basket-price-compare`

**Created**: 2026-10-03

**Status**: Draft

**Input**: User description: "MVP of Grocery Price Compare as described in requirement.md: nationwide Pakistan basket price comparer using only online prices, 300 staples, city-filtered stores, basket builder, store comparison with delivery fee/minimum order, cheapest one- and two-store split, unit prices, Urdu/English search, price history, freshness labels, WhatsApp sharing, admin match review, source health alerts, daily price collection from at least 5 chains."

**Source document**: `requirement.md` (Grocery Price Compare — Product Plan, Oct 2, 2026). Section references below (for example "MVP deliverables #6") point to that file.

## Clarifications

### Session 2026-10-03

- Q: If a store's terms of service forbid automated price reading, what should happen? → A: Skip that store; if fewer than 5 permitted stores remain, stop and ask the product owner before launch.
- Q: How should prices older than 24 hours count in basket totals and the cheapest-store suggestions? → A: Count them in store totals with a stale mark, but exclude stale prices when picking the cheapest store or split.
- Q: Should search also find items typed in Roman Urdu, such as "atta", "cheeni" or "ghee"? → A: Yes. Each item gets English, Urdu-script and Roman Urdu names, and all three are searchable.
- Q: Who sets each store's delivery fee and minimum order, and how? → A: Read them automatically from store sites.

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Compare my basket across stores in my city (Priority: P1)

A shopper opens the app, picks their city, adds the staples they buy each month with
quantities, and sees what the whole basket costs at each online store that delivers to
that city. Each total includes that store's delivery fee and shows whether the basket meets
the store's minimum order. Items a store does not sell are flagged. Every price is labelled
as an online price with the time it was last updated.

**Why this priority**: This is the product's wedge ("cheapest month of groceries", requirement
§Competitor landscape). Without it nothing else has value.

**Independent Test**: Load a fixed set of stores, cities, catalog items and prices. Pick a
city, add items, and check each store total against a hand-calculated expected value.

**Acceptance Scenarios**:

1. **Given** a first-time visitor, **When** they open the app, **Then** they are asked to pick
   a city before seeing any comparison.
2. **Given** a user picked Lahore, **When** they return later on the same device, **Then**
   Lahore is still selected and they can change it.
3. **Given** a user in Lahore, **When** they view a basket comparison, **Then** only stores
   that deliver to Lahore appear, and the screen shows how many stores deliver there.
4. **Given** a user searches "cooking oil" in English, **When** results appear, **Then** each
   result shows its pack size and its price per kg, litre or piece.
5. **Given** a basket of 3 items with quantities, **When** the user opens the comparison,
   **Then** each store shows the sum of (price × quantity) plus its delivery fee (0 if the
   subtotal reaches the store's free-delivery threshold), and the
   stores are sorted from cheapest to most expensive total.
6. **Given** a store that does not sell one of the basket items, **When** the comparison
   shows, **Then** that store's total is marked incomplete and names the missing items.
7. **Given** a basket total below a store's minimum order, **When** the comparison shows,
   **Then** that store is marked "below minimum order" with the amount still needed.
8. **Given** a user without an account, **When** they build a basket and leave, **Then** the
   basket is still there when they come back on the same device.
9. **Given** any displayed price, **When** the user views it, **Then** it reads "online price,
   updated <time>"; a price older than 24 hours is also marked stale.

---

### User Story 2 - Daily online price collection (Priority: P2)

Prices for the catalog are read from the public online catalogs of at least 5 chains. The
300 basket items refresh several times a day; each store's full catalog refreshes daily.
Every reading is kept with its time, and with its city for stores that price by city.

**Why this priority**: Story 1 needs real, current prices to be useful to anyone outside a
test.

**Independent Test**: Point collection at recorded copies of each store's catalog pages and
check that prices are stored with timestamps, that older readings are kept, and that a
store's request rate stays within its limit.

**Acceptance Scenarios**:

1. **Given** at least 5 configured stores, **When** a scheduled collection run finishes,
   **Then** each store has a new price reading for every listing it still sells, stamped with
   the reading time.
2. **Given** a product whose price changed since the last run, **When** the new price is
   stored, **Then** the earlier reading is still available.
3. **Given** a store flagged as pricing by city, **When** collection runs, **Then** one price
   per delivery city is stored for that store, each tagged with its city.
4. **Given** a store page that is behind a login, **When** collection is configured, **Then**
   that page is not read.
5. **Given** a run that fails for one store, **When** the run ends, **Then** the other stores'
   prices are still saved and the failure is recorded with the store and reason.

---

### User Story 3 - Admin reviews uncertain product matches (Priority: P3)

The same product appears under different names at each store ("Dalda Cooking Oil 5L
Bottle" vs "DALDA OIL 5 LTR"). The system links store listings to one master product,
automatically when it is certain and through an admin when it is not. Only verified links
ever reach shoppers.

**Why this priority**: Wrong matches destroy trust (requirement §Product matching). Story 1
cannot be trusted at scale without it.

**Independent Test**: Feed listings with known barcodes, known equivalent names and known
look-alikes of different sizes. Check which are linked automatically, which go to review,
and that unreviewed links never appear in a comparison.

**Acceptance Scenarios**:

1. **Given** a store listing with the same barcode as a master product, **When** matching
   runs, **Then** the listing is linked automatically.
2. **Given** a listing whose brand, product, size, unit and pack count all equal a master
   product's, **When** matching runs, **Then** it is linked automatically.
3. **Given** a listing that is similar but not certain, **When** matching runs, **Then** it
   goes to the review queue with the suggested master product and the reason.
4. **Given** an item in the review queue, **When** an admin approves it, **Then** its price
   appears in shopper comparisons; **When** an admin rejects it, **Then** it never does.
5. **Given** two listings of the same brand in different sizes, **When** matching runs,
   **Then** they are never linked to the same master product.
6. **Given** a non-admin visitor, **When** they try to open the review screen, **Then**
   access is refused.

---

### User Story 4 - Cheapest store and cheapest two-store split (Priority: P4)

For a basket, the app names the cheapest single store and the cheapest way to split the
basket between two stores ("buy these 12 at Carrefour, these 8 at Naheed"), counting each
store's delivery fee and minimum order.

**Why this priority**: Shows real savings and is easy to share, but builds on Story 1.

**Independent Test**: Use a fixed basket and fixed prices where the best split is known by
hand. Check both suggestions and the stated savings.

**Acceptance Scenarios**:

1. **Given** a basket and the stores in the user's city, **When** the user opens the
   suggestions, **Then** the cheapest single store that sells every item and meets its
   minimum order is named with its total.
2. **Given** the same basket, **When** a two-store split is cheaper after both delivery fees,
   **Then** the split is shown with the item list for each store, the combined total, and
   the saving versus the cheapest single store.
3. **Given** a split where one store's share is below its minimum order, **When**
   suggestions are calculated, **Then** that split is not offered.
4. **Given** no two-store split beats the single-store option, **When** suggestions show,
   **Then** the app says the single store is cheapest and offers no split.
5. **Given** a store whose price for a basket item is more than 24 hours old, **When**
   suggestions are calculated, **Then** that item counts as unavailable at that store, while
   the store's total in the comparison still includes it and is marked stale.

---

### User Story 5 - Use the app in Urdu or English (Priority: P5)

A shopper can switch the whole interface between Urdu and English and find any of the 300
items by typing its name in English, Urdu script or Roman Urdu.

**Why this priority**: Local fit (requirement §Features) and a key difference from
LowPrice.pk, but Story 1 works without it.

**Independent Test**: For every one of the 300 items, search by its English, Urdu-script
and Roman Urdu names and check it is returned in the results.

**Acceptance Scenarios**:

1. **Given** the interface is in English, **When** the user switches to Urdu, **Then** all
   interface text appears in Urdu with right-to-left layout, and the choice is remembered.
2. **Given** any of the 300 items, **When** the user types its Urdu-script name, **Then** the item
   appears in the results.
3. **Given** any of the 300 items, **When** the user types its English name, **Then** the
   item appears in the results.
4. **Given** any of the 300 items, **When** the user types its Roman Urdu name (for example
   "cheeni" for sugar), **Then** the item appears in the results.
5. **Given** a search with no matches, **When** results show, **Then** the app says nothing
   matched and suggests checking the spelling or trying the other language.

---

### User Story 6 - See a product's price history (Priority: P6)

A shopper opens a product and sees a line of its price at each store since tracking began,
so they can tell whether a "sale" is really a discount.

**Why this priority**: Answers job-to-be-done #2 ("Is this sale actually a discount?").

**Independent Test**: Load a known series of readings and check the chart shows each point
per store with its date.

**Acceptance Scenarios**:

1. **Given** a product with readings from 3 stores, **When** the user opens its history,
   **Then** one line per store shows price over time from the first reading to the latest.
2. **Given** a product with a single reading, **When** history opens, **Then** the app shows
   that one price and says tracking started on that date.
3. **Given** a store priced by city, **When** a user views history, **Then** the line uses
   that store's price for the user's city.

---

### User Story 7 - Share a basket on WhatsApp (Priority: P7)

A shopper shares their basket and its store comparison to WhatsApp as a link or an image.

**Why this priority**: Free distribution (requirement §Features), after the core works.

**Independent Test**: Share a basket, open the link on a different device, and check the
same items and comparison are shown.

**Acceptance Scenarios**:

1. **Given** a basket with a comparison, **When** the user taps "Share on WhatsApp",
   **Then** WhatsApp opens with a message containing the cheapest store, its total, and a
   link to the basket.
2. **Given** a shared link, **When** someone opens it without an account, **Then** they see
   the basket items and the store comparison, and can copy the basket to their own device.
3. **Given** a basket, **When** the user chooses "Save as image", **Then** an image with the
   items, the per-store totals and the price update time is produced.

---

### User Story 8 - Source health alerts (Priority: P8)

The admin is told when a store's collection looks broken, so bad data does not reach
shoppers unnoticed.

**Why this priority**: Protects data quality; the pipeline runs without it but less safely.

**Independent Test**: Feed collection runs that return no items, a 45% price jump, and a
25% smaller catalog, and check one alert is raised for each.

**Acceptance Scenarios**:

1. **Given** a store run returns 0 items, **When** the run ends, **Then** the admin gets an
   alert naming the store.
2. **Given** a product's price changes by more than 40% from its previous reading, **When**
   the reading is stored, **Then** the admin gets an alert naming the product, store, old
   price and new price.
3. **Given** a store's catalog is more than 20% smaller than the previous day's, **When** the
   daily run ends, **Then** the admin gets an alert with both item counts.
4. **Given** an alert was raised, **When** the admin opens the admin area, **Then** the alert
   is listed with its time and can be marked as handled.

---

### Edge Cases

- A user's city has no delivering stores: the app says no online store delivers there yet
  and lists the nearest supported cities.
- A city has only one delivering store: the comparison shows that store alone and no split
  is offered.
- A basket item is out of stock at a store: it counts as missing at that store.
- The basket is empty: the comparison screen explains how to add items instead of showing
  totals of zero.
- Every store is missing at least one item: no complete single-store option exists, and the
  app says so while still showing each store's partial total and missing items.
- Two stores have exactly equal totals: both are shown as cheapest.
- A store's listing changes pack size (5L to 4.5L): the old link is not reused; the new
  listing goes through matching again.
- A price jump over 40% is real (a genuine price rise): the price is still shown, and the
  alert lets the admin confirm it.
- The user changes city with a basket already built: the basket is kept and recompared for
  the new city's stores.
- A shared basket link points to items no longer in the catalog: those items are listed as
  no longer available.

## Requirements *(mandatory)*

### Functional Requirements

**City and stores**

- **FR-001**: System MUST let a user pick a city from the list of cities that at least one
  store delivers to, and MUST remember it on that device.
- **FR-002**: System MUST keep, for each store, the list of cities it delivers to, its
  delivery fee, and its minimum order value (per city where a store sets them by city).
- **FR-002a**: System MUST read each store's delivery fee, minimum order and any
  free-delivery threshold from the store's public site at least once a day, and keep each
  reading with its time, like prices (FR-010).
- **FR-002b**: If a store's delivery fee or minimum order cannot be read, the system MUST use
  the last successful reading and mark it stale after 24 hours. If no reading exists, that
  store's total MUST say "delivery fee unknown", and the store MUST be left out of
  cheapest-store and split suggestions.
- **FR-003**: Comparisons MUST include only stores that deliver to the user's selected city,
  and MUST show how many stores that is.

**Catalog, matching and prices**

- **FR-004**: System MUST hold a master catalog of 300 packaged staple products. Each store
  listing links to at most one master product; a master product can have many listings.
- **FR-005**: System MUST link listings automatically only by identical barcode, or by
  identical brand, product, size, unit and pack count. All other candidate links MUST go to
  an admin review queue.
- **FR-006**: System MUST NOT show shoppers any price from a listing whose link is not
  verified (automatic exact match or admin-approved).
- **FR-007**: System MUST NOT link listings of different sizes or pack counts to the same
  master product.
- **FR-008**: System MUST show each product's price per kg, per litre, or per piece next to
  its pack price.
- **FR-009**: System MUST collect prices from the public online catalogs of at least 5
  chains, reading the 300 basket items at least 3 times a day and each store's full catalog
  at least once a day.
- **FR-010**: System MUST store every price reading with its reading time, and its city for
  stores flagged as pricing by city. Readings MUST never be overwritten or deleted.
- **FR-011**: Collection MUST read only pages that need no login, MUST respect a per-store
  request rate limit, and MUST skip any store marked as not permitted. A store whose terms
  of service forbid automated reading MUST be marked not permitted. If fewer than 5
  permitted stores remain, launch MUST stop until the product owner decides.
- **FR-012**: Every price shown MUST be labelled "online price, updated <time>". A price
  whose latest reading is more than 24 hours old MUST also be marked stale. Stale prices
  count in store totals (FR-017), and a total that includes any stale price MUST be marked
  stale.
- **FR-013**: The app MUST state plainly that in-store shelf prices may differ.

**Search**

- **FR-014**: Users MUST be able to find any of the 300 products by its English name, its
  Urdu-script name, or its Roman Urdu name (Urdu written in English letters, for example
  "atta", "cheeni").

**Basket and comparison**

- **FR-015**: Users MUST be able to add products to a basket, set a whole-number quantity of
  1 or more, change it, and remove items, without creating an account.
- **FR-016**: The basket MUST persist on the user's device between visits.
- **FR-017**: For each delivering store, the comparison MUST show the sum of price ×
  quantity for the items it sells, plus its delivery fee, sorted from lowest to highest.
- **FR-018**: The comparison MUST flag, per store, which basket items it does not sell or
  has out of stock, and MUST mark that store's total as incomplete.
- **FR-019**: The comparison MUST mark any store whose basket subtotal is below its minimum
  order, with the amount still needed.

**Suggestions**

- **FR-020**: System MUST name the cheapest single store that sells every basket item and
  meets its minimum order.
- **FR-021**: System MUST find the cheapest assignment of basket items across any two
  delivering stores, counting both delivery fees and requiring each store to meet its
  minimum order, and MUST show it only when it is cheaper than the cheapest single store.
- **FR-021a**: When picking the cheapest single store or two-store split, an item whose
  price at a store is stale MUST be treated as unavailable at that store.
- **FR-022**: Each suggestion MUST show its item list per store, its total, and its saving
  versus the cheapest single store.

**Price history**

- **FR-023**: Each product MUST have a price history view showing one line per store from
  the first reading to the latest.

**Language and sharing**

- **FR-024**: The full interface MUST be available in English and Urdu, with right-to-left
  layout for Urdu, and the choice MUST be remembered on the device.
- **FR-025**: Users MUST be able to share a basket and its comparison to WhatsApp as a link,
  and save it as an image.
- **FR-026**: A shared link MUST open for anyone without an account, show the basket and
  comparison, and let the viewer copy the basket to their own device.

**Admin**

- **FR-027**: An admin area MUST be available only to signed-in admins.
- **FR-028**: Admins MUST be able to list uncertain matches, see the listing beside the
  suggested master product, and approve or reject each.
- **FR-029**: System MUST alert the admin when a store's run returns 0 items, a product's
  price changes by more than 40% between readings, or a store's catalog shrinks by more than
  20% from the previous day.
- **FR-030**: Admins MUST be able to see collection failures and alerts with their time, and
  mark alerts as handled.
- **FR-031**: Admins MUST be able to set, per store, its delivery cities, whether it prices
  by city, and whether collection is permitted. Delivery fees and minimum orders are read
  automatically (FR-002a), not entered by admins.

**Out of scope for this feature** (requirement §MVP deliverables): price-drop alerts, deals
feed, affiliate or "order on store" links, app-only stores (pandamart, inDrive.Groceries),
user accounts beyond saved baskets, fresh produce, in-store prices, receipt scanning, own
delivery or checkout.

### Key Entities

- **City**: A place a user can select. Has an English and an Urdu name.
- **Store**: An online grocery chain. Has its delivery cities, delivery fee, minimum order,
  free-delivery threshold if any (all read from its site, with reading time), a flag for
  pricing by city, a flag for whether collection is permitted, and a request rate limit.
- **Master product**: One canonical staple (for example Dalda cooking oil, 5 L, 1 pack). Has
  brand, product type, size, unit, pack count, barcode if known, English name, Urdu-script
  name, Roman Urdu name.
- **Store listing**: A product as a store sells it. Has the store's own name, barcode if
  given, parsed size and unit, and a link to one master product with its match status
  (automatic, pending review, approved, rejected).
- **Price reading**: One observed price for a store listing at a time, with the city when the
  store prices by city, and an in-stock flag.
- **Basket**: A user's list of master products with quantities, kept on their device and
  copyable via a share link.
- **Shared basket**: A snapshot of a basket reachable by link.
- **Collection run**: One attempt to read a store's catalog, with start and end time, item
  count, and failure reason if any.
- **Alert**: A health warning about a store or product, with type, details, time and
  handled status.
- **Admin**: A person allowed into the admin area.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: A first-time user can pick a city, add 10 items and see the store comparison in
  under 3 minutes on a phone.
- **SC-002**: The comparison for a 40-item basket appears within 2 seconds of opening it on a
  typical mobile connection.
- **SC-003**: At least 5 chains have prices updated within the last 24 hours on 95% of days.
- **SC-004**: 100% of the 300 catalog items are found by their English, Urdu-script and
  Roman Urdu names.
- **SC-005**: In an audit sample of 200 links shown to shoppers, at least 98% are correct
  (same product, same size, same pack count).
- **SC-006**: 100% of displayed prices carry an update time, and every price older than 24
  hours is marked stale.
- **SC-007**: Hand-checked baskets match the app's store totals and split suggestions
  exactly in 100% of test cases.
- **SC-008**: Each of the three health conditions raises an alert within one collection run
  of occurring.
- **SC-009** (launch signal, from requirement §MVP deliverables): 300 waitlist signups
  before launch, and 25% of users still building baskets in week 4.

## Assumptions

- **A-001**: Prices are one national price per product per store unless a store is flagged as
  pricing by city. *Source*: requirement §Stores to cover, "Pricing model".
- **A-002**: The MVP stores are Carrefour, Al-Fatah, Naheed, Metro, Chase Up, GrocerApp and
  Daraz grocery; at least 5 must be live, counting only stores whose terms permit automated
  reading (see Clarifications). *Source*: requirement §Daily price collection, MVP.
- **A-003**: Delivery fee, minimum order and free-delivery threshold are read automatically
  from each store's public site (see Clarifications). When the basket subtotal reaches a
  store's free-delivery threshold, its delivery fee counts as 0. *Source*: requirement
  §Features, "Delivery fee and minimum order included"; product owner answer 2026-10-03.
- **A-016**: When a store's delivery fee or minimum order cannot be read, the last good
  reading is used, then marked stale after 24 hours, matching the price freshness rule
  (FR-012). *Reasoning*: keeps fees and prices under one freshness rule. *Source*:
  requirement §Stores to cover, "Price label".
- **A-004**: The price used is the price a customer pays now on the store's site, including
  any sale price shown. *Reasoning*: the comparison answers "cheapest today". *Source*:
  requirement §MVP deliverables, summary line.
- **A-005**: "Several times a day" for basket items means at least 3 runs a day.
  *Reasoning*: the smallest number that is clearly more than daily. *Source*: requirement
  §Daily price collection.
- **A-006**: Baskets are saved on the user's device only. Moving a basket to another device
  is done through a share link. *Reasoning*: "without signing up" (requirement MVP
  deliverables #5) rules out server accounts.
- **A-007**: "Urdu and English" means both the interface and product names, with a language
  switch. *Source*: requirement §Features, "Urdu and English".
- **A-008**: Urdu-script and Roman Urdu names for the 300 products are written by hand as part of building the
  master catalog. *Source*: requirement §Product matching, "300 hand-verified basket items".
- **A-009**: Fuzzy text and image similarity scoring (requirement §Product matching, step 3)
  is not used for automatic linking in this MVP; everything that is not an exact barcode or
  exact attribute match goes to admin review. *Reasoning*: with 300 items and about 50
  reviews a day, review is manageable and avoids wrong automatic links. Fuzzy scoring may
  still rank suggestions in the review queue.
- **A-010**: Admins are a small, named set of people who sign in. Shoppers never sign in.
  *Source*: requirement §MVP deliverables #11 and "user accounts beyond saved baskets" out
  of scope.
- **A-011**: Admin alerts appear in the admin area and are also sent by email. *Reasoning*:
  "the admin is alerted" (requirement MVP deliverables #12) implies notice outside the app.
- **A-012**: The two-store split is limited to exactly two stores; three or more is out of
  scope. *Source*: requirement MVP deliverables #7.
- **A-013**: A shared link shows a snapshot of the basket items; prices on the shared page are
  the current ones when it is opened, with their update times. *Reasoning*: stale shared
  prices would mislead. *Source*: requirement §Stores to cover, "Price label".
- **A-014**: The pre-build check (requirement §Stores to cover, "Before building": 10
  products in Lahore vs Islamabad for Carrefour and Al-Fatah) is a manual research task
  done during planning, and its result sets each store's "prices by city" flag.
- **A-015**: The cities list comes from the stores' delivery lists; "ships nationwide"
  stores deliver to every listed city.
