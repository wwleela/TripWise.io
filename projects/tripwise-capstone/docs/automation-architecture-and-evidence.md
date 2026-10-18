# TripWise automation architecture and test evidence

## Current wiring
**Scenario:** TripWise — Itinerary Review, Make scenario ID 8009532, My Team, EU1. Scenario is active. Existing scenario reused; no replacement scenario created.

| Order | Module | Responsibility | Data contract / guardrail |
|---:|---|---|---|
| 1 | Custom Webhook | Accept a save-itinerary event from Supabase | Opaque itinerary_event_id; destination, duration, traveller type/count, pace, budget, estimate, over-budget flag, title; persona_key added in save-itinerary v7. No email or account ID. |
| 2 | Data Store — ExistRecord | Check idempotency key | Data Store 222515; event ID is the key. Existing event must be filtered out before AI/logging. |
| 3 | Groq chat completion | Generate one internal review sentence | Connection 12030148, model openai/gpt-oss-20b, max_tokens 500, temperature 0.1. It may only use supplied metadata and must disclose that activity fit cannot be evaluated if activity details are not supplied. |
| 4 | Airtable Search Records | Resolve persona deterministically | Connection 12030093; base appYkjF5PDb1ZOyJd; Personas table tblOr6qV8OHPjE5U0; formula matches incoming persona_key against stable Persona Key field. No fuzzy or model-based match. |
| 5 | Airtable Create Record | Create internal review queue record | Itinerary_Log table tblZWPvV41V4Hyfyb; Persona receives the returned Airtable record ID as a linked-record value; Review Status = Pending Review. |
| 6 | Data Store — AddRecord | Record processed event only after Airtable write | Data Store 222515; key = event ID. |
| 7 | Webhook Respond | Acknowledge completion to Supabase | JSON response status=logged and the same event ID. Supabase save-itinerary now requires both before setting explanation_log_status=logged. |

## Credentials and secrets
- Supabase project: uwbhcqywmgcosyqwmmlq.
- Server-side Edge Function secret name: MAKE_WEBHOOK_URL. Do not put the webhook URL or Groq API key in the repository or frontend.
- Groq connection and Airtable OAuth connection are authorized in Make. No keys or tokens are stored in this document.
- Review explanation remains internal. Do not expose it in a traveller-facing UI, API response, itinerary detail page, dashboard or export.

## Real observed test transcripts

### Test 1 — Goa sample while webhook was in learning mode
**Input:** Goa, 4 days, group of 4, relaxed pace, total budget ₹15,000, estimated cost ₹47,000, over_budget=true, title “Goa — 4 days”.
**Observed output:** Make accepted the HTTP delivery at 2026-10-18 04:40:15 UTC and learned the payload schema. There was no scenario execution for this event because the webhook was still in learning mode.
**Verdict:** real webhook contract sample, not a completed automation run.

### Test 2 — Darjeeling, Groq blank output
**Input:** Darjeeling — 2 days; solo; 1 traveller; relaxed; budget ₹15,000; estimated cost ₹5,212; over_budget=false.
**Observed output:** Make execution completed five module operations in about 3.5 seconds. Groq output result was an empty string and finish_reason was length under the old 120-token cap. Airtable record rec4zPx6wFTbkOry5 was created with a blank AI Explanation; the event key was added to the Data Store.
**Verdict:** flow technically completed but the content-quality assertion failed. Corrective action: increased Groq token cap to 500 and required an explicit processing acknowledgement rather than treating generic webhook acceptance as success.

### Test 3 — Pondicherry, nonempty explanation
**Input:** Pondicherry — 3 days; solo; 1 traveller; relaxed; budget ₹15,000; estimate ₹8,600; over_budget=false.
**Actual Groq output:** “This 3-day solo, relaxed-paced trip to Pondicherry has a total budget of 15,000 with an estimated cost of 8,600, which is within budget, but the suitability of the actual activities cannot be assessed from the supplied metadata alone.”
**Observed output:** five Make modules completed without errors in about 4.1 seconds. Airtable created record recLjYPv8mafQ5OHN, Review Status Pending Review; Data Store stored event ID 861e05ed-5c7a-475c-8d0f-042b18711bed.
**Verdict:** nonempty internal review record proven, but this was before the new persona-link mapping and acknowledgement contract. It is not evidence that the new linked Persona logic is passing.

## Important negative evidence and next test

Three Supabase trips were directly observed in the database: Goa, Darjeeling and Pondicherry. The two full Make executions observed before the latest persona-link patch are Darjeeling (blank explanation) and Pondicherry (nonempty explanation). The Goa webhook delivery was a learning-mode sample only. Do not claim three complete automation passes.

The new save-itinerary function version 7 adds persona_key:
- family → family_trip_coordinator
- other supported traveller types → self_serve_value_planner

It also checks the Make response body before marking logging successful. The main Make scenario now performs a Persona search and passes the returned Airtable record ID to the linked Persona field. These changes are deployed/configured but require a fresh app save to verify as a single flow.

**Release test, to run next**
1. Save one fresh supported-destination itinerary in the app.
2. Inspect the latest Make execution; confirm the Persona search returns one real record.
3. Open the created Airtable row and click Persona to confirm it opens the exact linked record.
4. Confirm the explanation is non-empty and Review Status is Pending Review.
5. Confirm the Supabase trip changes to explanation_log_status=logged only after Make responds with the same event ID.
6. Replay the same event ID in a controlled safe test and verify no second Airtable record is created.
7. Check failure path: save must survive if Make/Groq/Airtable fails; logging state must not falsely become logged.

## Human review checkpoint

The Make/Groq sentence is a review artifact, not a customer output. Airtable is the internal queue. A reviewer may mark Approved, Rejected or Reviewed, but the current MVP does not display the explanation in the traveller app even after approval. Approval visibility changes are explicitly out of scope until a separate approval-status integration is designed and tested.

