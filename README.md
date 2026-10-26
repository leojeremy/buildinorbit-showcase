# Build in Orbit

Build in Orbit lets a small merchant in Kenya or the United States open an online store: a storefront at their own address, a catalogue, checkout, orders, delivery, taxes, and payments that go straight into the merchant's own accounts. Merchants pay the platform a subscription. It is built on Laravel 13, Livewire 4 and PostgreSQL.

A solo project: I own the design and every decision below. This repo is the engineering thinking behind it. The two shops it was designed around sell differently, get paid differently and charge tax differently, and most of the decisions below came from serving both without building two products.

## Decisions

**Many shops, one codebase, no leaks between them.**

Every store's data lives in the same database, tagged with the store it belongs to. The risk is one shop seeing another's orders. The rule I set is that no business action may look up "the current store" from the environment: the store is always passed in explicitly. That turns a leak into something you can see in the code instead of something hiding in a global.

*Trade-off:* more to pass around, and every new feature has to follow the rule.

**Two customers can't buy the last item at once.**

Stock can only change in one place, inside a database transaction that locks the row, and an architecture test fails the build if anything else writes to it. My first tests ran on SQLite, which ignores row locks, so they would have passed even with the protection broken. I added PostgreSQL-only tests that check the lock query really runs before the stock changes.

*Trade-off:* nothing may update stock directly, even where a quick shortcut would be easier.

**Never trust what a merchant types.**

Stores publish their own text and links, so everything a merchant writes is treated as untrusted on its way to the page. I keep that handling in one place instead of scattering it through templates, which means it can be reviewed and tested as a unit rather than hoped about.

*Trade-off:* merchants get less formatting freedom than a free-form editor would give them.

**Getting paid is different in every market.**

In Kenya most customers pay by M-Pesa to a till or paybill. The US shop takes Cash App and Zelle. Instead of hard-coding a list of gateways, every payment is a row in a ledger and every way of paying has the same shape. "Paid", "balance" and "due now" are worked out from the ledger, so part payments, pay later and sending a customer a payment link all come from the same model, and a new way of paying is one more entry rather than a new flow.

*Trade-off:* more upfront work than wiring one gateway, and totals are derived instead of read from a field.

**Not every merchant needs every feature.**

Features are modules. What a store sees depends on its country, its plan, an override from the platform and the merchant's own choice, and the result is enforced on the server: switching a feature off makes its pages a 404, not just a hidden menu item.

*Trade-off:* each gated route has to be registered against its module.

**Strict by default.**

Eloquent strict mode and lazy-loading prevention are on outside production, so a missing eager load fails loudly in development and CI instead of becoming a slow page. Key endpoints have query-count tests, for example the cart's item count must cost no query at all for a visitor without a cart.

*Trade-off:* development is stricter, and a quick prototype needs tidying before it passes.

## The source
The source is private. I'm happy to walk through any part of it in an interview, or to give named interviewers read-only access.

[LinkedIn](https://linkedin.com/in/jeremymleo) · jeremymleo@gmail.com
