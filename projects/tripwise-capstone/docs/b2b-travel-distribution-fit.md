# TripWise — B2B Travel Distribution / Wholesale Fit Assessment

**Status:** Exploratory product hypothesis only. Separate from the current TripWise capstone MVP and not implemented in the published app.  
**Decision date:** 18 October 2026.

## Executive decision

Do **not** bolt a generic “Wholesale” tab onto the current consumer TripWise prototype.

The existing product is a B2C, budget-aware itinerary planner. It intentionally uses a bounded catalogue, labels costs as estimates and keeps live prices/bookings out of scope. Wholesale travel distribution is a different business workflow: a travel agent or tour operator obtains trade/net rates from suppliers, assembles a sellable quote for an end traveller, checks dates/availability/terms, applies a commercial margin and manages supplier confirmation.

There is a credible market category: TBO describes itself as a B2B travel-distribution platform and reports connecting 159,000+ travel buyers in 100+ countries with millions of travel suppliers. This is evidence that B2B travel distribution is an established model—not proof that TripWise's current users want this feature or that a new product can compete in it.

Source: [TBO — Investor/company profile](https://www.tbo.com/engagement/investors/)

**Recommendation:** keep the consumer TripWise product stable. Validate a separate **TripWise Partner Desk** concept with independent travel agents and local tour operators before implementing supplier net rates, inventory or bookings.

## Product fit contrast

| Dimension | Current TripWise | Potential B2B travel-distribution track |
|---|---|---|
| Primary user | Traveller planning their own trip | Independent travel agent / tour operator preparing a quote for a client |
| Main job | Plan a trip inside a budget and supported catalogue | Source supplier products, verify availability, assemble a quote with a viable margin and obtain supplier confirmation |
| Price model | Indicative, catalogue-grounded estimates | Supplier-provided net/contracted price, fees/taxes and agent sell price/markup |
| Inventory | No live inventory or booking claim | Must include date/quantity availability and freshness, or clearly require supplier confirmation |
| Transaction | Save/revisit itinerary | Request/confirm quote, record supplier response and track booking state |
| Failure cost | An inaccurate estimate can mislead a traveller | An incorrect net rate, unavailable product or hidden cancellation term can erase margin or cause a booking failure |
| Launch decision | Preserve current scope | Validate in a separate discovery track; do not mix with existing capstone success metrics |

## Customer problem hypothesis

An independent travel agent may coordinate hotels, transfers, activity operators and package suppliers across email, WhatsApp, spreadsheets and distributor portals. They may manually convert supplier net costs into a customer-facing quote, check availability and re-confirm terms. This is a hypothesis, not a verified research finding for TripWise.

### Discovery before build

Interview 5–8 independent travel agents and 3–5 accommodation/activity/transfer suppliers. Ask for the last real quote and booking workflow:
1. How did you request and confirm the net rate and availability for specific travel dates?
2. Which fees, taxes, child rates, inclusions, blackouts or cancellation terms are commonly missed?
3. How do you calculate and protect agency margin across a multi-component itinerary?
4. How often does the supplier's initial quote differ from the final accepted rate?
5. Which channels/systems must remain in the workflow?
6. Would the supplier share an up-to-date list in a structured format, and how often can it be confirmed?

Do not build from imagined agents or use the consumer traveller persona as a proxy for travel-agent needs.

## Narrow prototype hypothesis

If discovery shows a painful quote-preparation workflow, prototype a single destination and one supplier category first.

### Minimum supply record

- Supplier identity and contact/confirmation channel.
- Product/experience identifier, destination and exact variant.
- Applicable date range, season/blackout dates and last-confirmed timestamp.
- Net rate, currency, taxes/fees, inclusions/exclusions and rate basis (per person, room, group, vehicle, etc.).
- Capacity/availability state, cutoff time and minimum party/booking conditions.
- Cancellation, amendment, no-show and refund terms.
- Supplier confirmation requirement and quote validity.
- Permission/contract that governs which agents may access the rate.

### Agent workflow

1. Supplier rate list is uploaded/imported or manually entered with source and validity.
2. Agent chooses destination/date/party size and assembles supplier-backed components.
3. System rejects incompatible dates/party sizes and flags stale or unknown rates.
4. Agent sees net cost, known fees and calculated margin privately; the customer-facing quote shows the approved sell price and its inclusions.
5. Agent requests confirmation from supplier.
6. Only on explicit supplier confirmation does the item become confirmed/bookable in the workflow.
7. Quote and accepted-rate snapshots are stored so later supplier price changes do not rewrite historical economics.

Use CSV/manual confirmation for the first pilot. Do not require expensive API integrations before the workflow is validated.

## Guardrails

- No supplier price is shown as current unless its validity/freshness is known.
- No inventory is labelled available without a trusted, time-scoped signal or explicit supplier confirmation.
- No quote can claim a booking is confirmed until the supplier confirms it.
- Do not expose net rates, commission or supplier-only terms to the traveller.
- Keep quote snapshots immutable after sending; later revisions create a new version.
- Separate agent, supplier and platform-operator permissions; test that an agent cannot read another agent's negotiated rates or client records.
- Display currency, per-unit basis, tax treatment, inclusions/exclusions and cancellation terms clearly.
- Add idempotency to supplier confirmation and any future booking event; retries must not create duplicates.

## Pilot success metrics

| Metric | Definition |
|---|---|
| Quote preparation time | Median time from agent starting a request to sending a complete quote |
| Supplier response time | Median time from request to explicit supplier confirmation/rejection |
| Rate accuracy | Share of accepted bookings whose final accepted supplier rate matches the quoted snapshot, subject to defined exclusions |
| Availability mismatch | Share of requests marked available that a supplier later rejects for capacity/date reasons |
| Quote-to-book conversion | Bookings confirmed / eligible quotes sent, reported by destination/category/sample window |
| Gross margin visibility | Share of sent quotes with validated net cost, charges and sell-price margin calculated |
| Rework rate | Quotes revised due to missing fees, incorrect terms or stale rates / quotes sent |
| Repeat-agent activity | Agents completing another quote or booking within the chosen cohort window |

Set targets only after observing a baseline and pilot sample. Gross margin should be calculated from confirmed contract terms and actual invoices, not assumed savings.

## Architecture implication

Do not reuse Katastima's procurement schema as-is. Both products can share engineering principles—source provenance, freshness, commercial-term visibility, immutable transaction snapshots, idempotency and role-scoped access—but their entities and economics differ.

- **Katastima:** retailer buys stock from wholesalers; core entities are supplier offers, schemes, purchase orders, stock receipts and cost/inventory effects.
- **TripWise Partner Desk:** a travel agent sources services at net rates and quotes a traveller; core entities are suppliers, rate plans, date-specific availability, itinerary/quote versions, margin, confirmation and cancellation terms.

Share implementation utilities only after both schemas and workflows are proven. Do not force one generic `wholesale_products` table to represent grocery cartons and travel experiences.

## Go / no-go gates

**Proceed to clickable prototype only if:**
- At least 5 agents independently describe the same costly quote/confirmation task.
- At least 3 suppliers can provide date-/term-aware rates for a small test catalogue.
- A clear confirmation channel exists and product-data permissions are understood.
- Manual pilot economics can be measured without promising live booking inventory.

**Proceed to production integration only if:**
- Supplier confirmation and cancellation states can be reliably observed.
- Net rate and customer sell-price confidentiality passes permission tests.
- Golden tests cover date validity, unknown availability, fees, currency, margins, cancellations, partial confirmations and retry/idempotency.
- Agents demonstrate use and a measurable reduction in quote rework/time.

If these gates fail, keep TripWise as the consumer itinerary tool and focus on budget integrity; do not add a poorly supported B2B layer simply because wholesale distribution is a sizeable category.
