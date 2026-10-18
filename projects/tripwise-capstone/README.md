# TripWise — Budget-Safe AI Trip Planner

**An AI Product Management capstone for the MakeMyTrip AI Trip Assistant case.**

TripWise is a working prototype for Indian travellers: authenticate, choose a supported destination, generate a catalogue-grounded day-by-day plan, see an INR estimate and explicit over-budget warning, and save/retrieve trips. A separate Make.com workflow generates a concise evidence-constrained review note via Groq and stores it in Airtable for **internal human review only**.

> **Product truth:** deployed itinerary generation is deterministic and catalogue-based, not LLM-generated. Groq is used in the internal review workflow. The review explanation is never exposed in the traveller-facing UI. Do not describe the current prototype as using an LLM to generate itineraries.

## Live artefacts

- **App URL:** `[ADD THE PUBLISHED BOLT APP URL]` — not independently captured in this package; insert and verify before submission.
- **Airtable base:** https://airtable.com/appYkjF5PDb1ZOyJd — access must be set to “Anyone with the link can view” for assessors, then re-tested in a signed-out/incognito window.
- **Automation:** Make scenario `8009532`, “TripWise — Itinerary Review”.
- **Repository folder:** https://github.com/wwleela/wwleela.github.io/tree/main/projects/tripwise-capstone
- **Single graded submission:** copy `CAPSTONE_SUBMISSION.md` into one Google Doc, add the verified app and Airtable links, set “Anyone with the link can view”, and submit that one Google Doc URL. GitHub is portfolio evidence, not a substitute for the required Google Doc.

## System at a glance

| Layer | Implementation | Verified status |
|---|---|---|
| Web app | Bolt / React | Build and typecheck passed in the latest reported build; app URL needs insertion |
| Authentication and persistence | Supabase project `uwbhcqywmgcosyqwmmlq` | Live tables and saved trips verified |
| Catalogue | 10 destinations, 52 activities | Live table counts verified |
| Itinerary logic | Deterministic selection from curated catalogue; server-side total recomputation | Source inspected; real Goa, Darjeeling and Pondicherry trips saved |
| Internal AI review | Make + Groq `openai/gpt-oss-20b` | Two scenario executions inspected; one blank response exposed a token-limit issue; later run produced non-empty text |
| Human review log | Airtable `Itinerary_Log` | Real rows created; both deterministic Persona lookup branches returned the intended Airtable record IDs in read-only QA. A fresh app-save test is still required to verify the linked-record write end to end. |
| Idempotency | Make Data Store `222515` | Event keys stored on successful prior runs; explicit same-event replay test still required |

## Current live evidence

- **Goa — 4 days:** group of 4; budget ₹15,000; estimated cost ₹47,000; over-budget warning true. The webhook arrived during learning mode, so this is not counted as a completed Make execution.
- **Darjeeling — 2 days:** budget ₹15,000; estimate ₹5,212. Make executed, but Groq returned an empty string at the former 120-token cap. This failure was preserved as evidence and the token budget was raised.
- **Pondicherry — 3 days:** budget ₹15,000; estimate ₹8,600. Make completed 5 modules; Groq produced a non-empty explanation; Airtable created record `recLjYPv8mafQ5OHN`. This record predates Persona-link routing, so its Persona link is not evidence of the new mapping working.
- The next fresh save is required to verify the updated `persona_key`, live Persona lookup, real Airtable linked record, and Make response acknowledgement.

## Portfolio contents

- `CAPSTONE_SUBMISSION.md` — canonical four-part document draft
- `docs/automation-architecture-and-evidence.md` — scenario structure and actual test transcripts
- `docs/responsible-ai-and-monitoring.md` — specific guardrails, bias mitigations and monitoring
- `docs/pricing-gtm-growth.md` — price, positioning, VPC, funnel and growth loop
- `docs/product-decisions-and-analytics.md` — product decision log, event taxonomy, metric formulas and release gates
- `docs/research-sources-and-evidence.md` — research source register and evidence limitations
- `evals/golden-dataset-and-scorecard.md` — 8-case dataset, rubric, assertions, and honest run status
- `linkedin-launch-post.md` — recruiter-oriented post draft

## Non-negotiable reporting rule

Untested means **Not tested**. Proto-personas and RICE confidence are hypotheses until primary research validates them. Never claim eight golden cases passed, public sharing is enabled, an app URL works, or Persona linking is complete without fresh evidence.
