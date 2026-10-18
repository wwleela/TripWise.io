# TripWise — Budget-Safe AI Trip Planner

**AI Product Management capstone** for the MakeMyTrip AI Trip Assistant brief.

## Live artefacts
- **Published app:** https://ai-trip-planner-web-7ogc.bolt.host
- **Airtable research and review base:** https://airtable.com/appYkjF5PDb1ZOyJd
- **GitHub project folder:** https://github.com/wwleela/TripWise.io/tree/main/projects/tripwise-capstone
- **Master capstone manuscript:** https://github.com/wwleela/TripWise.io/blob/main/projects/tripwise-capstone/CAPSTONE_SUBMISSION.md
- **LinkedIn draft:** https://github.com/wwleela/TripWise.io/blob/main/projects/tripwise-capstone/linkedin-launch-post.md

## What this project demonstrates
- Product discovery and explicitly labelled proto-personas.
- RICE prioritisation, PRD-lite, INVEST user stories and precise metrics.
- A published, catalogue-grounded trip planner with authentication and saved itineraries.
- Make → Groq → Airtable internal explanation review queue.
- Actual automation transcripts, eval rubric, an eight-case golden dataset and responsible-AI monitoring plan.
- Pricing hypotheses, GTM, Value Proposition Canvas, funnel analysis and lifecycle plan.

## Product truth
The traveller-facing itinerary is generated deterministically from a curated catalogue; it is not LLM-generated. Groq is used only for a separate internal reviewer note, which remains Pending Review and is never exposed in the traveller-facing MVP. Live prices and bookings are out of scope.

## Verified evidence and gaps
Three saved trips were observed in Supabase: Goa, Darjeeling and Pondicherry. A previous Make test produced a blank Groq explanation at a 120-token limit; a later test produced a nonempty, appropriately caveated explanation and an Airtable row. Read-only Make tests confirmed that the family Persona lookup returns Family Trip Coordinator and the solo lookup returns Self-Serve Value Planner.

Synthetic webhook QA now passes: the straight-through path created a Pending Review record with a genuine linked Persona; replaying the same event returned the explicit logged/event-ID acknowledgement without creating a second row. This does not prove the published app’s save-itinerary function has fired the latest payload; one fresh app-generated save remains a release gate. The eight golden app eval cases are specified but not yet run.

## Repository contents
- CAPSTONE_SUBMISSION.md — complete four-part master manuscript
- docs/automation-architecture-and-evidence.md — automation structure and observed transcripts
- docs/responsible-ai-and-monitoring.md — safeguards and bias monitoring
- docs/pricing-gtm-growth.md — pricing, GTM, VPC, revenue scenario and funnel
- docs/product-decisions-and-analytics.md — decision log, event taxonomy and release gates
- docs/research-sources-and-evidence.md — source register and evidence limitations
- evals/golden-dataset-and-scorecard.md — eight cases, rubric, assertions and honest run status
- linkedin-launch-post.md — recruiter-oriented post draft

## Required academic submission
The assessed deliverable is one Google Doc URL only. Copy the master manuscript into one Google Doc, set “Anyone with the link can view,” and submit that single Google Doc URL. The app link belongs in Part 2 and the Airtable link belongs in Part 1. GitHub is supplemental portfolio evidence.
