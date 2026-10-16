# Product decisions

[Product overview](README.md) · [Architecture](ARCHITECTURE.md)

These decisions describe the service at the end of Sprint 2, **16 October 2026**: a free cost calculator, limited free launch reports, bilingual educational content and consent-aware measurement.

## Start with the host's own economics

A market average cannot tell a host whether a short stay covers cleaning, laundry and commission. The calculator therefore establishes two nightly rates: one covers entered costs, the other includes the requested profit before income tax. Booking totals, payout and profit are named separately.

A separate cleaning fee is guest revenue while cleaning remains an expense. Keeping both visible supports comparison of short and long stays without hiding costs. Numerical results use deterministic formulas.

## Let a visitor request a report directly

Some hosts want local market context before completing a cost calculation. The direct report path asks for geography and property parameters, then confirms the email address. It does not invent financial inputs or force the visitor through the calculator.

The calculator-to-report path remains available and carries the host's actual inputs and selection forward. Both paths lead to the same saved report model.

## Bound the launch offer

The current offer is capped at 50 report grants, with one grant per verified email and municipality. Atomic reservation protects the cap under simultaneous requests. Failed and uncertain requests keep their reservation until reviewed.

A repeat opening uses the stored snapshot rather than spending on another commercial API request. Changing the original selection must not silently change a previously granted report.

The former €5 purchase path is retained behind explicit configuration, with paid sales disabled. Enabling payments is a separate product decision; the public documentation must describe the offer a visitor can actually use.

## Give each source a distinct job

AJPES describes registered establishments, SURS supplies municipal tourism observations, and AirROI supplies commercial historical market estimates. They support the same decision but describe different populations and units.

Source dates, retrieval dates and definitions travel with the report. Missing or suppressed values remain visible. A daily source check cannot make the latest published monthly tourism data more recent.

## Make geography explicit

The visitor chooses an official municipality and a compatible commercial market. The resolver rejects unrelated or unconfirmed combinations. A supported locality relationship does not establish identical administrative boundaries; that uncertainty is stated in the report.

Free official-data coverage across 212 municipalities does not imply equivalent commercial coverage. The system retains exact provider identifiers and the version of the geographic rule used.

## Preserve the report independently of email

Entitlement, report readiness and email delivery are separate states. The customer can open a ready report when its email fails. Sending the same result link again does not refetch the market data.

Atomic claims prevent normal duplicate processing. An uncertain supplier-call or mail outcome requires review instead of a blind retry. This is a controlled recovery strategy, not a promise of exactly-once behaviour across external services.

## Separate confirmation and consent

Email confirmation establishes control of an address and resumes the requested result. Marketing consent is optional and has its own recorded state and withdrawal path.

Analytics consent and advertising measurement consent are separate from transactional delivery. A visitor may receive the report without joining a marketing list. Private links, emails and personal cost inputs do not belong in public campaign URLs or creative assets.

## Measure materials through to a ready report

A bounded registry of twelve campaign links distinguishes organic Facebook/LinkedIn posts and Google/Meta ad variants. Separate campaign values are the simplest working attribution mechanism for this release; unsupported free-text URL values are sanitized.

Article assistance remains separate from the acquisition source. Duplicate confirmation and report reopening do not count as additional ready reports.

Owner/friend tests are separated when interpreting demand. Search indexing, working instrumentation and a technical advertising handoff were the Sprint 2 goals; traffic growth and paid-campaign results require subsequent real use.

## Publish bilingual content through one catalogue

The marketer supplies EN/SL text, sources, metadata, an image and localized alternative text. The developer registers the translated pair, places the image, adds internal links and calls to action, and validates both pages.

The catalogue generates the sitemap and language relationships, reducing the chance of a published article being omitted. Search Console is checked after publication; deployment itself does not prove indexing.

## Keep operations independently observable

A daily cloud task refreshes and publishes official data. A separate hourly task checks freshness, failed or stuck report/mail states, quota evidence and resource limits.

Incident and recovery notices use Gmail, independently of the transactional EmailJS channel. Unchanged checks stay quiet. An enabled schedule or silent inbox is not proof of a successful run; dated execution and delivery evidence are retained privately.

## Keep the public showcase accurate

This repository describes the deployed service and separates disabled paths, verified behaviour and pending commercial decisions. Archived screenshots are dated rather than presented as the current interface.

Application source, customer data, credentials and private operational evidence remain outside the public showcase. Documentation changes do not grant a new software licence or claim customer adoption.
