# TripWise responsible-AI design and monitoring

## Product principle
A compelling itinerary is not automatically an accurate or affordable itinerary. TripWise prioritises catalogue grounding, deterministic cost recomputation, clear uncertainty, owner-scoped data and an internal-only AI reviewer queue. The current itinerary generator is deterministic. Groq is only used by the internal Make workflow, not to generate the traveller’s itinerary.

## Input handling and prompt injection
- Traveller interests are mapped to fixed categories. They are not sent to an itinerary-generating LLM.
- Database queries use destination_id and curated catalogue tables rather than concatenating free-text into SQL.
- Make’s Groq system prompt treats event data as untrusted, says to ignore embedded instructions, restricts the model to supplied fields and prohibits invented prices, attractions, availability and reviews.
- Negative test requirement: put an instruction-like payload in interests and confirm it cannot change rules, expose secrets or bypass data validation. The app eval for this is currently Not tested.

## Hallucination and uncertainty
- Itinerary activities come from the curated activities table and have catalogue IDs and provenance fields.
- Costs are indicative, not live quotes. Server-side code recomputes category/day totals instead of trusting a client total.
- Unsupported destinations produce a clear limitation rather than invented hotels/fares.
- The internal review prompt now explicitly states that it cannot assess suitability of the day-by-day activities when the webhook contains only metadata. This was observed in the Pondicherry transcript.
- Residual limitation: the current webhook gives only trip metadata, so a one-line explanation cannot meaningfully assess the exact activity sequence. A next iteration could add a bounded, privacy-safe activity summary, then relearn the Make webhook schema and re-run tests.

## PII and privacy by design
| Data | Is it collected? | Safeguard |
|---|---|---|
| Authentication email | Auth provider handles sign-in | Not sent to Make or stored in the internal explanation event. |
| Authenticated user ID | Stored on owned trip record | Enforced by owner-scoped RLS; not sent to Make. |
| Starting city | City-level text in trip preferences | Do not request precise location; avoid sending it to the LLM review payload unless a concrete purpose requires it. |
| Dates and destination | Required trip planning fields | Keep inside the user's saved trip; webhook includes destination/duration metadata only. |
| Traveller type/count, interests, pace, budget | Required planning preferences | Keep payload minimal and never include direct identifiers. |
| Passport, card details, exact GPS, government identifiers | Not collected | Explicitly out of scope. |
| API keys/webhook URL | Server-side secrets only | Never commit to GitHub, frontend environment variables or logs. |

## Four bias risks and concrete mitigations

### 1. Popularity bias
Mitigation: the current deterministic generator does not rank by click volume or popularity. It prioritises user interest categories and rotates through catalogue categories. Monitor the concentration of top recommended destination/activity IDs. Residual: a biased catalogue can still recreate popularity bias; assess the catalogue itself before claiming it is solved.

### 2. Price bias
Mitigation: compute totals and compare with the user's stated budget, display cost notes and issue a visible over-budget warning. Do not assume expensive means better. Test low-budget and over-budget cases. Residual: cost assumptions are catalogue estimates, not live price guarantees.

### 3. Geographical bias
Mitigation: do not silently substitute unsupported destinations; disclose the supported-catalogue boundary. Catalogue expansion should include a region-coverage audit before release. Residual: the current catalogue has only ten destinations; geographical balance is not yet proven.

### 4. Review bias
Mitigation: the deterministic generator does not consume ratings or review count as ranking features. Recommendations are sourced from catalogue metadata, category and interests. Residual: since no review data is used, the prototype avoids review-volume dependence but cannot claim to evaluate review fairness in a live reviews system.

## Govern / Map / Measure / Manage

### Govern
Product owner sets intended outcomes and approves rubric/guardrail changes. Engineering owns access controls, total recomputation and integration reliability. A reviewer handles the internal Airtable queue. A production launch requires tested RLS, actual linked Persona, nonempty explanations, all eight golden cases scored, and an observed duplicate-replay test.

### Map
Risks include unsupported destinations, misleading cost assumptions, hidden overruns, prompt injection, overconfident explanation, popularity/price/geography/review bias, PII transfer, cross-user access, duplicate webhook events and accidental disclosure of internal explanation.

### Measure
Run the eight golden tests before every release, weekly during the first month of a pilot, monthly thereafter, and after every change to catalogue, ranking, cost logic, prompt or integration. Metrics: budget arithmetic mismatch, silent overrun, unsupported-destination rate, catalogue diversity, blank/failed Groq output, unmatched Persona records, duplicate log rate and logs incorrectly marked logged.

### Manage: geographical concentration resurfaces
- Detect: track unique destination IDs per 100 itineraries and top-three destination share. Trigger investigation if concentration exceeds a pre-agreed threshold for seven days.
- Contain: disable or constrain the affected recommendation rule while leaving saved trips available. Keep generated explanations in Make/Airtable internal review.
- Fix: inspect catalogue coverage/category rotation, adjust selection and rerun all eight golden cases. Review affected Airtable log entries and document the change.
- Communicate: record impact, time window, mitigation and retest outcome in the project change log; notify stakeholders and correct materially misleading customer-facing content if any was released.

## Known release gates
- Fresh save after the latest persona_key and webhook-ack patch: pending.
- Real linked Persona in the new log row: pending fresh confirmation.
- Duplicate event replay: pending.
- Eight app golden tests: not yet run.
- Public Bolt app URL and Airtable public view permissions: must be verified before final submission.

