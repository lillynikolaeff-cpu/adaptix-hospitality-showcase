# Product decisions

[Product overview](README.md) · [Architecture and integrations](ARCHITECTURE.md)

These decisions define the completed first release: a free cost calculator and a €5 local market comparison for independent hosts in Slovenia.

## Start with the host's own economics

A market average cannot tell a host whether a short stay covers cleaning, laundry and commission. The free calculator therefore establishes two complete nightly rates first: one covers the entered costs, the other includes the requested profit before income tax. Booking totals, payout and profit are named separately.

A separate cleaning fee is revenue from the guest. It changes the displayed nightly price while actual cleaning remains an expense. Making both visible lets hosts compare shorter and longer stays without hiding costs.

## Make the paid result answer a specific question

The €5 purchase adds a local reference to the host's calculation: the market price distribution, registered accommodation supply and tourism history for the selected area. One purchase has one municipality, property category and bedroom selection, with a dated snapshot and defined access period.

The compact page can be revisited and printed. Recalculating personal costs is free because it does not require fresh supplier data. A new market slice is a separate explicit purchase. This keeps the service understandable for occasional users and ties data spending to a paid request.

## Give each data source a distinct job

AJPES describes registered establishments, SURS provides municipal guest-night context, and AirROI provides commercial market estimates. They contribute separate blocks to a single decision; they do not measure the same population. A register entry is not a listing, a registered bed is not a bedroom, and a guest overnight stay is not a booked property-night.

The design imports official data into versioned aggregates and requests the commercial market slice after payment. Source dates and definitions travel with the result. Missing observations remain visible instead of turning into zeroes or invented estimates.

## Treat geography as part of what the customer buys

The host selects an official municipality. A reviewed crosswalk connects that choice to SURS and the exact AirROI market identifiers. A successful API response is only one check; the boundary must also match.

The service sells comparisons only for verified selections. It does not substitute a nearby town or a broader region without the customer's knowledge. The order pins the mapping version so a later correction cannot silently change what was purchased. Free coverage across all 212 municipalities can coexist with a smaller set of eligible paid markets.

## Preserve the result independently of email

Payment success, report readiness and message delivery are separate states. The customer can open a ready result even when its email fails. Resending the link reuses the saved result and does not consume another paid data call.

Duplicate payment notifications are expected and deduplicated. An uncertain supplier-call outcome needs reconciliation before a retry. The owner sees paid orders without results and can resolve them or issue a refund.

## Keep contact confirmation separate from marketing

Anyone can calculate without signing in. Email confirmation is required to receive a saved calculation and proceed to payment, while marketing consent remains optional. Confirmation proves control of an address; it does not create a subscription by itself. Consent and withdrawal are stored as versioned events, and current application state governs any later outreach.

## Use a small, operable architecture

One Cloudflare application and D1 database support the interface, customer workflow, source aggregates and owner operations. Isolated adapters handle each data contract. Scheduled imports stay outside customer requests, and durable order jobs keep fulfilment independent of browser sessions.

The owner uses a protected administration area and can export CSV to a private Google Sheet. Manual refund handling and CSV import are sufficient for the first release. Automatic spreadsheet synchronization, dynamic rate publishing, calendar integration and expanded analytics remain later product choices.

## Make the showcase assessable

The architecture describes the intended finished service. A separate dated checkpoint identifies delivery progress. Public screenshots illustrate the implemented interface; neither they nor this design establish customer adoption, revenue or conversion performance. Those require evidence from actual use after release.
