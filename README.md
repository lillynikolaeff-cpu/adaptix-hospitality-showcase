# Adaptix Hospitality

**Pricing and local market intelligence for independent accommodation hosts in Slovenia.**

[Product](https://adaptix-hs.eu) · [Architecture and integrations](ARCHITECTURE.md) · [Product decisions](PRODUCT-DECISIONS.md)

Adaptix Hospitality helps a host answer three connected questions: **What nightly rate covers my costs? How does my price compare with the local market? What do official data tell me about accommodation supply and tourism in my municipality?**

The product combines a free pricing calculator with a one-time **€5 market comparison**. It brings the host's own booking economics together with the AJPES accommodation register, SURS tourism statistics and AirROI market estimates, in a compact result that can be saved, emailed and reopened.

> This showcase describes the completed first-release product and its target architecture. A dated implementation checkpoint appears at the end; features described here are the delivery scope, not a claim that every integration is already live.

## Who it serves

An independent host renting a room, apartment or holiday home, often as a source of additional income. The host needs an understandable price calculation and a local reference without maintaining a spreadsheet or subscribing to a professional revenue-management platform. Slovenian is the primary language, with an English interface alongside it.

## The product experience

### 1. Calculate a sustainable booking price — free

The host enters the length of the stay, cleaning and laundry costs per booking, nightly running costs, monthly fixed costs, expected occupied nights and a target profit. Editable commissions support direct bookings, Airbnb and Booking.com.

The result shows two complete alternative prices: **Break-even rate** for the entered costs, and **Rate with profit**. Each includes a booking total, channel fees, payout and profit before income tax. A separate cleaning charge can be added without removing the actual cleaning expense from the calculation.

The host can also select any of Slovenia's 212 municipalities to see a dated count of active AJPES register entries in the accommodation categories relevant to this product. The calculation and this limited supply indicator are available without registration.

### 2. Save the result

An optional email flow confirms ownership of the address before sending the calculation. The confirmation link returns to the saved inputs and result. Marketing consent is a separate, optional choice with an unsubscribe path.

### 3. Compare with the local market — €5 once

The host selects a municipality, accommodation category and bedroom count. Before payment, the site shows the exact comparison scope and checks that the selected municipality has a reviewed AirROI match. After email confirmation and Stripe payment, the host receives a result with three parts:

| Part of the result | What the host learns | Data behind it |
| --- | --- | --- |
| **My price and the market** | How the calculated nightly rate sits alongside a historical market median and range, with the reference period and listing count | Host inputs + AirROI estimates for the selected market and property filters |
| **Registered accommodation supply** | How many active establishments are registered locally, their types and registered capacity | AJPES Register nastanitvenih obratov (RNO) |
| **Tourism context** | How monthly guest overnight stays vary and how a period compares with the same period a year earlier | SURS SiStat municipal tourism statistics |

The result opens on the site and is linked in an email. One purchase covers one geographic and property selection and one market snapshot. The intended access period is up to 30 days within supplier retention limits. Reopening that snapshot and changing the host's own costs are free; a new market snapshot requires a new explicit purchase.

## How the data becomes a useful comparison

**AJPES** supplies a daily ZIP containing the accommodation register in XML. An import validates the snapshot and groups active establishments by their actual municipality, type and registered capacity. Only the defined free count is public; the paid result adds the detailed breakdown. [AJPES download documentation](https://www.ajpes.si/Registri/Drugi_registri/Register_nastanitvenih_obratov/Splosno)

**SURS** supplies a monthly municipal series through the SiStat PxWeb API. The integration selects guest overnight stays, preserves missing or suppressed observations and builds the seasonality context from published periods. [SURS table 2164466S](https://pxweb.stat.si/SiStatData/pxweb/en/Data/-/2164466S.px/)

**AirROI** supplies filtered historical market estimates through a server-side API request after verified payment. The result retains its geography, filters and reference months, with attribution. [AirROI API](https://www.airroi.com/api/documentation)

A reviewed mapping connects official municipal codes with the supplier's exact market identifiers. The system assembles these sources at the area level; it does not attempt to identify individual properties across databases. Each block keeps its own definition, date and coverage. The full [architecture](ARCHITECTURE.md) explains extraction, validation, storage, refreshes and failure handling.

## Built to operate as a small service

The owner has a protected workspace for contacts, orders, delivery and source health. It shows whether an address is confirmed, whether marketing consent exists, whether payment succeeded, whether the result is ready and whether the email provider accepted the message. CSV exports support a private Google Sheets workflow. Data freshness, geographic review status and supplier-call failures are visible alongside customer operations.

The application uses **React, TypeScript, Vinext and Vite**, deployed to **Cloudflare Workers through OpenAI Sites**. **Cloudflare D1 and Drizzle** support application state and normalized data. **Stripe** handles payment, **EmailJS** sends transactional messages, and platform-managed sign-in plus a server-side owner check protects administration.

The first release delivers pricing guidance and a purchased market snapshot. Automatic rate publishing, booking-calendar synchronization, subscriptions and custom scraping of accommodation websites are outside that release.

## Interface

The screenshots show the implemented free calculator. The architecture above also covers the paid result and integrations delivered by the end of the project.

![English free calculator with booking costs and two nightly rates](assets/calculator-en.png)

<details>
<summary>Slovenian mobile calculator</summary>

![Slovenian calculator and booking result on mobile](assets/calculator-sl-mobile.png)

</details>

## Implementation checkpoint — 7 October 2026

The free calculator, limited AJPES municipality count, confirmed-email delivery and owner contact view are live. AJPES import rules and initial AirROI response validation are established. Paid checkout and fulfilment, the recurring source pipelines, SURS result integration and the full owner order/data views remain delivery work. A successful AirROI response alone does not enable a municipality for purchase: boundary review is also required.

## About this showcase

Product design and development by **Lilly Dubovik / Adaptix**, using ChatGPT and Codex as development tools. Pricing calculations are deterministic; AI does not generate the numerical result.

This repository presents the product, system design and public interface. Application source, customer records, credentials and raw supplier datasets are kept private.
