# Architecture and integrations

[Product overview](README.md) · [Product decisions](PRODUCT-DECISIONS.md)

**Implemented architecture at 23 October 2026 (version 67).** The public service offers a free calculator and a capped launch-report flow. The application also contains a separately gated Stripe payment path. Paid sales are disabled at this checkpoint.

## System map

```mermaid
flowchart TB
    UPDATE["Daily cloud update: 07:00 Europe/Budapest"] --> AJPES["AJPES ZIP / XML"]
    UPDATE --> SURS["SURS SiStat API"]
    AJPES --> PAIR["Validate and atomically prepare official-data.json"]
    SURS --> PAIR
    PAIR --> DEPLOY["Build and publish source revision through Sites"]
    DEPLOY --> APP["React / TypeScript / Vinext application on Cloudflare Workers"]
    HOST["Host: EN or SL"] --> APP
    APP --> CALC["Deterministic pricing calculation"]
    APP <--> DB[("D1: contacts, requests, grants, reports, mail and analytics")]
    APP --> MAP["Versioned geographic resolver and data-quality checks"]
    MAP --> AIRROI["Authorized, filtered AirROI request"]
    AIRROI --> APP
    APP --> EMAIL["EmailJS transactional email"]
    APP --> METRICS["Consent-aware GA4 / Meta events"]
    OWNER["Authenticated owner"] --> ADMIN["Contacts, reports and analytics administration"]
    ADMIN --> DB
    MONITOR["Independent hourly cloud task"] --> DEPLOY
    MONITOR --> DB
    MONITOR --> QUOTA["Verified quota and balance snapshots"]
    MONITOR --> GMAIL["Gmail incident and recovery notices"]
```

Official data is currently shipped as a validated JSON bundle in the published source revision. Customer and workflow state is stored in D1. There is no claim that official municipal series live in separate production D1 import tables, or that an independently scheduled queue worker generates customer reports.

## Official-data refresh

The daily cloud task downloads AJPES and queries SURS outside visitor requests. It validates the pair, commits the generated source, builds and publishes through Sites. A local atomic rename prevents one source being activated without its paired dataset; a successful deployment determines which bundle the live application serves. Source dates, retrieval dates and completion time remain distinct.

### AJPES

The [RNO export](https://www.ajpes.si/RNO/RNO.zip) contains registered accommodation establishments. The importer uses the establishment municipality, rather than the operator's postal address, selects active records and aggregates accommodation types and registered capacity.

The bundle covers all 212 municipalities. Beds, accommodation units and unknown capacity remain distinct measures. The public free count is a narrower projection than the report's detailed supply context. Raw register rows and operator details are not public report payloads.

### SURS

[SiStat table 2164466S](https://pxweb.stat.si/SiStatData/pxweb/en/Data/-/2164466S.px/) provides municipal tourist arrivals and overnight stays. The importer resolves dimensions from metadata, queries the published periods and retains missing/suppressed observation flags.

The latest published month may be older than the retrieval date. Comparable-period calculations require available observations and a valid denominator. Guest overnight stays describe the statistical table's population, not property occupancy in the commercial sample.

A failed refresh must retain the last valid published pair. The data-quality checks assess age and completeness separately from the schedule being enabled.

## Geography and AirROI

A versioned catalogue retains exact AirROI market identifiers, official municipality association, boundary status and review evidence. The resolver checks the selected market and its relationship to the municipality. Broader or otherwise non-exact coverage must be labelled explicitly; the service does not silently present a different territory as an exact municipal match.

AirROI requests run on the server with property-type and bedroom filters and a historical reference window. The adapter validates response geography, months, values and distribution ordering. Missing or inadequate coverage is reported rather than replaced with a fabricated estimate.

The price block identifies historical nightly-rate ADR, excludes separate cleaning fees and describes values before host commission. It is not a forecast or the total guest bill. Monthly distributions retain their periods; averages of monthly statistics must not be labelled an annual median.

Reports store the derived result, official context, geography version, quality warnings and generation date. Private access expires after 30 days; reopening does not reset the deadline or make a new AirROI request. External-provider rights and retention requirements continue to govern the data. [AirROI API documentation](https://www.airroi.com/api/documentation)

## Hotel history and published offers

The monthly country-wide AirROI collection covers hotel and boutique-hotel listings declared for two guests. The October snapshot contains 100 listings across 43 provider areas, with positive observed prices in 35 areas during the historical window. These are coverage observations, not a census of Slovenian hotels. A standard room category is not established by these filters.

Derived monthly statistics are equally weighted across eligible listing ADRs. A single observation is labelled as one observation, not a market range. Small samples carry a caution; absent prices leave an explicit gap while other report sections remain available. Customer report requests read the stored hotel cache rather than purchasing the national collection again.

The existing weekly radar supplies published offers from official hotel websites. Its October snapshot has 16 sources and 93 accommodation-offer records after filtering. From-prices, packages, per-person rates, dates and source links retain their meaning; expired and stale offers are hidden. They are not pooled into the historical hotel median.

A personal price comparison requires a real saved calculation and a valid comparable reference. Direct market reviews do not invent a personal quote. Entire homes and hotel rooms remain different products with a visible explanation.

## Direct report and calculator-to-report paths

```mermaid
sequenceDiagram
    actor Host
    participant App as Application
    participant Mail as EmailJS
    participant DB as D1
    participant Market as AirROI
    Host->>App: Select area and property; optional calculation
    App->>DB: Save request and hashed confirmation token
    App->>Mail: Send confirmation link
    Host->>App: Confirm email, including on another device
    App->>DB: Resolve request and atomically grant launch report
    App->>DB: Claim pending fulfilment
    App->>Market: Fetch validated, eligible market selection
    Market-->>App: Historical estimates
    App->>DB: Persist report, provenance and access expiry
    App->>DB: Record report_ready once where consent allows
    App->>Mail: Send ready-report link
    App-->>Host: Open stored report
```

Direct requests store the selection without inventing calculator inputs. Calculator-originated requests preserve the actual inputs and selected municipality. Confirmation resumes the persisted request, so a different device does not depend on the first browser's local state.

The launch grant is unique per verified email and municipality. An atomic database operation enforces the global 50-report limit, including failed or uncertain reservations. A repeat request reuses the grant and its original selection.

Fulfilment is an application function protected by an atomic database claim. Request handling and Cloudflare `waitUntil` where used protect execution against an early browser disconnect; durable states make interrupted work detectable. This is not a guarantee of an independently running job queue. Unknown billable-call outcomes are held for review instead of blindly retried.

Report readiness, grant/payment status and email delivery are separate. Provider acceptance is recorded as acceptance, not proof of inbox delivery. A failed email does not erase a ready report. Resending an eligible ready-result link reuses the stored report; unknown send outcomes require reconciliation.

## Retained payment path

The application contains a €5 Stripe Checkout path, guarded by runtime configuration and an explicit sales flag. It requires a confirmed contact and authorized saved calculation. Signed payment events are checked against order identity, mode, currency, amount and pinned geography.

Repeated events and concurrent claims are deduplicated. Failed fulfilment has a recorded refund path; uncertain supplier and refund outcomes remain visible. The existence and testing of this code do not mean live paid sales are enabled. The current public launch report is granted at zero charge and does not invoke Stripe.

## Persistent state and access

| Stored group | Purpose |
| --- | --- |
| Contacts and consent events | Address verification, language, optional marketing consent and withdrawals |
| Free results and report requests | Persisted inputs or selections, confirmation state and continuation |
| Orders / launch grants | Immutable entitlement, selected market, fulfilment and delivery states |
| Saved report payloads | Derived market result, source context, warnings and expiry |
| Mail attempts and locks | Shared quota reservations and controlled transactional sending |
| Analytics events and resource context | Deduplicated journey events, attribution, consent and internal-test classification |
| Stripe events and feedback | Payment-event deduplication and report feedback |

D1 is accessed through server code and Drizzle-managed schema/migrations. Owner administration uses platform sign-in plus server authorization. Customer report access requires a valid session or authorized expiring link; knowing a record identifier alone is insufficient.

Confirmation secrets are hashed and expire. Rate limits, same-origin checks, payload validation and bounded quota reservations protect the public flows. Credentials and raw provider payloads are not published in this showcase.

## Search, articles and attribution

Article content is stored as an EN/SL JSON pair with shared translation identity, localized metadata, sources and images. Registration in the article catalogue drives the indexes and generated sitemap. The page template supplies canonical/language links, Open Graph and Article structured data.

Public article pages return complete server-rendered text and localized metadata, clean self-canonical URLs and reciprocal language URLs. The fixed `/go/bled-en` and `/go/bled-sl` aliases return an HTTP 302 to their allowlisted article and campaign. Query input cannot replace the destination. Unknown aliases return 404. Internal links and CTA paths do not add UTM parameters.

The report landing pages have localized public share cards. Private result pages are excluded from public discovery.

Sixteen allowlisted material links use distinct `utm_campaign` values for organic posts and paid variants. Unknown values are sanitized; `utm_content` and `utm_term` are not retained. This bounded approach distinguishes materials through the ready-report event without accepting arbitrary URL text into analytics. Article assistance is retained as a separate context.

GA4 browser and server events respect analytics consent. Meta measurement requires its separate advertising consent. Duplicate confirmation and reopening do not emit another ready-report conversion. Owner/friend tests are identified separately when interpreting demand; a nominally external browser session is not automatically a real lead.

## Operations and independent monitoring

The existing main task runs hourly at minute 00. It runs the watchdog, gates the daily official-data publication to the morning window, prepares the 08:00 digest, and runs the monthly hotel stage only when due. A separate hourly task at minute 03 checks the service. Actual start times depend on the cloud queue.

| Process | Cadence and responsibility |
| --- | --- |
| AJPES/SURS | Daily 07:00 local publication of the validated pair |
| Independent monitor | Hourly minute 03; service freshness, reports, mail, resources and incidents |
| Watchdog | Hourly minute 00; missing, stalled or incomplete monitoring |
| Daily digest | 08:00 Europe/Budapest; previous 24 hours, including zero orders and unresolved incidents |
| Hotel cache | 20th of the month, 07:00 Europe/Ljubljana; first paid scheduled cycle on 20 November |
| Published-offer radar | Existing weekly review and validated import |

Owner-only tools persist runs, incident transitions, notifications, resource snapshots and digest states in D1. The operations dashboard uses these records and distinguishes source import from a verified deployment. City or district names are readable and colored dots accompany written statuses.

Incident/recovery messages and the digest use connected Gmail independently of transactional EmailJS. Deduplication and atomic claims prevent routine repeats. A send with an unknown outcome requires reconciliation; acceptance by a provider is not inbox-delivery proof. The digest stores separate Sent and Inbox evidence after verification.

On 23 October, the official update published version 66 autonomously, the service checks and watchdog succeeded, and the daily zero-order digest was verified at 08:03 with one attempt and one message. Version 67 then added the article pair and first short links. Full delivery identifiers remain private.

Resource balances are dated observations of shared accounts. Forecasts use up to seven comparable days and identify insufficient history, resets and one-off research spending. Automatic balance retrieval from provider APIs is not yet confirmed.

The hotel stage requires a balance observation no older than 24 hours, enforces a $15 monthly limit and $5 client-report reserve, and does not buy a completed month again. A failed refresh keeps the usable prior cache and does not block the main report. November's first paid cycle remains a future verification.

## Build and validation boundary

The private application uses Node.js 22.13 or newer, pnpm, React, TypeScript, Vinext/Vite, Cloudflare Workers, D1 and Drizzle. The repository's lockfile pins its build inputs.

The maintained deployment workflow is: retrieve the Site's source; install its locked dependencies; run the relevant unit/runtime checks, TypeScript and production build; synchronize the verified source revision; package and publish through Sites; verify the deployed result. Runtime secrets are configured through Sites and are not placed in source-control examples.

At the Sprint 3 checkpoint, 179 tests, TypeScript and build passed, together with built-Worker integration checks and 16 material-attribution journeys. Provider responses and outbound email/Meta calls in those runtime tests are intercepted. These checks verify software behaviour; they do not establish campaign performance or customer adoption.

This showcase is documentation-only and cannot be built or deployed as the application. A transfer of private application source, configuration and operating credentials is a separate repository handover.
