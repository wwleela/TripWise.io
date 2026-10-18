# TripWise — MakeMyTrip AI Trip Assistant Capstone

**Product:** TripWise — Budget-Safe, Catalogue-Grounded Day-by-Day Itinerary Planner  
**Role represented:** AI Product Manager  
**Stack:** Bolt + Supabase + Make + Groq + Airtable  
**Status date:** 18 October 2026

> **Submission instruction:** This is the working master manuscript, not the final graded submission. The course requires one Google Doc link only. Copy this complete manuscript into one Google Doc, keep the published app URL embedded in Part 2 below and the Airtable base URL in Part 1 §1.6, verify both open in a signed-out/incognito window, set the Google Doc to “Anyone with the link can view,” and submit that single Google Doc URL. GitHub is supplemental portfolio evidence and cannot replace the required Google Doc.

## Executive summary

TripWise is a working prototype for Indian travellers who need a day-by-day plan with visible cost assumptions and a clear budget guardrail. A signed-in user selects a supported destination, enters dates, traveller type/count, interests, pace and INR budget, generates a catalogue-grounded itinerary, saves it and retrieves saved trips later.

The product decision is deliberately narrower than a general-purpose travel chatbot: prioritise **trustworthy budget-aware itinerary structure** before adding live prices, bookings, maps or a more elaborate conversational assistant. The current itinerary generator is deterministic and uses a curated catalogue; the separate Make/Groq step produces an internal reviewer note. The app must not falsely imply that an LLM generated or fact-checked the traveller’s itinerary.

The internal explanation is retained in Airtable with a Pending Review status. It is not displayed in the traveller UI. This is a deliberate scope and safety decision for the MVP.

**Verified evidence as of 18 October 2026:** three app-saved itinerary records were observed in the live Supabase database (Goa, Darjeeling and Pondicherry). A prior app-triggered Pondicherry run produced a non-empty, caveated internal note; an earlier Darjeeling run exposed a blank Groq output and led to the token-limit correction. After the latest changes, synthetic webhook QA verified the non-empty Groq output, a genuine linked Family Trip Coordinator Persona, Pending Review status, matching `status=logged` acknowledgement, and duplicate-event replay without a second Airtable create. This is synthetic workflow evidence, not a fresh save from the published app. The eight golden app cases, public signed-out app access, public Airtable assessor link, cross-user isolation test and traveller-UI explanation test remain unverified release gates.

# Part 1 — Research, persona and feature PRD

## 1.1 Problem framing

The assignment describes a risk that travel recommendation systems can overweight popularity, high price, review volume and geographical concentration instead of individual needs. TripWise takes that scenario as the product challenge and tests a narrow countermeasure: structure the user’s preferences and total budget first, choose only from a small supported catalogue, show estimates and their assumptions, and warn when the computed total exceeds the requested budget.

This is a prototype hypothesis, not proof that a production MakeMyTrip algorithm has these measured biases. The exercise starts from the case brief and uses external public sources to understand trip-planning alternatives and domestic travel expenditure.

## 1.2 Proto-persona A — Primary: Self-Serve Value Planner

**Archetype and name:** Self-Serve Value Planner — an illustrative persona representing a budget-conscious Indian leisure traveller who wants to self-serve the planning task.

**Background:** Plans short domestic trips with a fixed overall spending limit. Uses digital booking and map tools but still has to stitch information across multiple pages and categories.

**Goals:** Convert destination, duration, interests and budget into a realistic sequence of daily activities; understand the total before committing; keep a plan that can be revisited.

**Behaviours:** Starts with a broad destination idea, checks multiple sources, adds up costs mentally, and adjusts duration or activities when the total becomes too high.

**Pain points:** Costs appear across accommodation, food, local transport and activities; a plan can look appealing while its total is not feasible; estimates can be mistaken for live quotes.

**Needs:** A clear daily plan, catalogue-backed options, visible category totals, provenance/assumption notes, an explicit over-budget warning and persistence after sign-in.

**Representative quote — synthetic, not an interview quote:** “Show me a plan that fits the money I have, and tell me what is an estimate before I start booking.”

**Segmentation tags:** Behavioural (self-serve planner), Value-based (cost conscious), Need-based (budget visibility), Geographic (domestic India).

**Expressed need:** Help me plan a trip within my budget.  
**Latent need:** Preserve decision confidence by making assumptions, uncertain costs and infeasible choices visible early, so that the traveller does not confuse an attractive itinerary with an affordable one.

**Empathy map**
- Think: “Can I fit the experiences I care about inside this total?”
- Feel: Worried about surprise spend; frustrated by cross-site research.
- Say: “Keep the budget in mind; don’t show me prices you cannot verify.”
- Do: Compare options, change duration, review totals and save the version that seems feasible.

**AEIOU**
- Activities: Search destinations, compare options, balance daily time and spending.
- Environment: Phone/laptop, booking sites, maps, saved notes and spreadsheets.
- Interactions: Filters, cards, activity lists, price estimates and saved trip views.
- Objects: Dates, destination, interests, catalogue activity, category estimates, total budget.
- Users: Solo traveller or small-party planner with limited time to reconcile sources.

**Contradiction/workaround:** The traveller asks for a simple answer, but still opens multiple tabs and performs manual arithmetic because a neat itinerary is not sufficient evidence that the entire trip fits the budget.

## 1.3 Proto-persona B — Family Trip Coordinator

**Archetype and name:** Family Trip Coordinator — an illustrative persona representing a person organising a shared domestic trip for a household.

**Background:** Coordinates different interests and energy levels across several people. Needs a manageable day structure and an understandable full-trip spending view rather than a list of disconnected popular attractions.

**Goals:** Balance family-suitable pacing, shared interests and cost; minimise backtracking and decision friction; share a clear plan.

**Behaviours:** Asks others what they want, collects destination ideas, tries to group nearby activities, and revises the itinerary when daily costs or timing appear unrealistic.

**Pain points:** More travellers multiply cost assumptions; a packed itinerary can be impractical; recommendations may serve the loudest or most popular preference rather than the whole group.

**Needs:** Clear pacing, category-based estimates, an obvious over-budget state, a supported destination catalogue and a reliable way to revisit the saved plan.

**Representative quote — synthetic, not an interview quote:** “Give us a realistic day plan everyone can understand, and show the complete spend before we commit.”

**Segmentation tags:** Demographic (household/family), Behavioural (multi-person decision coordinator), Value-based (household budget), Need-based (shared pacing).

**Expressed need:** Plan activities the family can do together.  
**Latent need:** Reduce coordination load and disagreement by making trade-offs among pace, variety and cost legible to the group organiser.

**Empathy map**
- Think: “Will this be manageable for the group and fit the full budget?”
- Feel: Responsible for others’ experience; anxious about hidden costs and rushed pacing.
- Say: “Keep the day realistic; don’t oversell activities as confirmed.”
- Do: Collect preferences, filter options, review day order and share a saved plan.

**AEIOU**
- Activities: Coordinate interests, assemble day plans, estimate per-person and shared costs.
- Environment: Family/group chat, mobile apps, booking pages and shared notes.
- Interactions: Forms, itinerary summaries, saved plans and cost breakdowns.
- Objects: Traveller count, dates, interests, pace, category totals and destination/activity catalogue.
- Users: Household or group organiser and the people whose needs they coordinate.

**Contradiction/workaround:** The organiser wants a low-effort plan, but manually consolidates opinions and costs because the apps used by the group do not automatically create one validated, budget-aware shared decision view.

**Evidence qualification:** Both personas are proto-persona hypotheses derived from the assignment and desk research, not validated interview personas. The Ministry of Statistics and Programme Implementation’s Domestic Tourism Expenditure Survey collects expense heads such as accommodation, food/drink, transport, shopping, recreation, religious, cultural, sporting and health-related activities. That supports treating category visibility as a useful question to test; it does not validate either persona individually. Source: https://www.pib.gov.in/PressReleasePage.aspx?PRID=2313894&lang=1&reg=3

## 1.4 Competitive teardown

This teardown uses publicly described features, not claims of a controlled hands-on product audit.

| Alternative | Type | Publicly described strengths relevant to the job | Table stakes / differentiators / delighters | TripWise implication |
|---|---|---|---|---|
| Wanderlog AI Trip Planner | Direct | AI travel assistance, trip itinerary, map, route optimisation, collaboration, budget tracking and expense splitting | Table stakes: itinerary and saved plan. Differentiators: route map, budget/splitting and collaboration. Delighter: trip organisation in one view. | Do not compete on feature breadth; focus on legible INR budgets and limitations for a supported catalogue. |
| Booking.com AI Trip Planner / AI property features | Direct | Conversational planning support; personalised property filters; property Q&A based on listing/reviews/photos; review summaries tested/introduced in selected markets | Table stakes: personalised suggestions. Differentiator: inventory-linked property answers. Delighter: property-specific Q&A that reduces search effort. | TripWise must not claim live inventory; use explicit estimates and unsupported-data handling. |
| Expedia trip-planning assistant / Trip Matching | Direct | Conversational travel planning and AI-supported transformation of travel inspiration into itineraries with booking pathways | Table stakes: recommendations and itinerary. Differentiator: tying inspiration into bookable inventory. Delighter: reel-to-itinerary discovery. | Narrow prototype scope to a reliable plan and budget guardrail; live bookability remains out of scope. |
| Google Maps saved lists / trip planning | Indirect | Save places and routes; organise and share place lists; directions and route planning | Table stakes: saved places and navigation. Differentiator: location context. Delighter: shareable lists as an easy planning artefact. | A future TripWise export could become a responsible AI Product Loop, but sharing is not in this MVP. |
| Manual search + notes/spreadsheet | Indirect | Flexible, no new product learning required, user can combine sources | Table stakes: user control. Differentiator: unrestricted source choice. Delighter: user-defined planning model. | TripWise’s core value is reducing manual cost reconciliation and making assumptions visible. |

Sources:
- Wanderlog AI planner, itinerary, budgeting, routing and collaboration: https://wanderlog.com/trip-planner-ai and https://wanderlog.com/home
- Booking.com AI Trip Planner and property Q&A / review summaries: https://news.booking.com/bookingcom-enhances-travel-planning-with-new-ai-powered-features--for-easier-smarter-decisions/
- Expedia conversational planning and Trip Matching: https://www.expedia.com/newsroom/expedia-launches-conversational-trip-planning-powered-by-chatgpt-to-inspire-members-to-dream-about-travel-in-new-ways/ and https://www.expedia.com/newsroom/now-live-expedia-launches-industry-first-feature-that-turns-reels-on-instagram-into-bookable-travel-itineraries/
- Google Maps shared lists: https://support.google.com/maps/answer/7280933?hl=en-IN

**MakeMyTrip SWOT for AI-assisted trip planning**
- Strengths: existing travel marketplace and booking funnel; publicly announced a multilingual GenAI Trip Planning Assistant (Myra) in August 2025 and an OpenAI collaboration to deepen AI-led discovery in February 2026. Sources: https://investors.mmtcdn.com/Press_Release_Gen_AI_Trip_Planning_Assistant_07_08_2025_404cc1872a.pdf and https://investors.mmtcdn.com/Make_My_Trip_Deepens_AI_First_Strategy_with_Open_AI_Collaboration_152a91fca9.pdf
- Weaknesses to test: recommendations can be difficult to trust if estimates, ranking logic and limits are hidden; the assignment frames concentration/popularity, price, geography and review bias as risks.
- Opportunities: improve conversion by making the plan explainable, budget-aware, inclusive of different trip styles and connected to action only where booking data is verified.
- Threats: competitors and general-purpose assistants can shorten planning; hallucinated prices or overly confident recommendations could damage trust; a polished itinerary can increase frustration if the final total is infeasible.

## 1.5 Four candidate ideas and RICE selection

Scale: Reach = relative reachable segment (1–10), Impact = expected user/business effect (1–5), Confidence = hypothesis confidence as decimal 0–1, Effort = estimated team-weeks. Scores are directional product judgments, not measured usage data.

| Candidate | Primary persona | Reach | Impact | Confidence | Effort | RICE arithmetic | Score | Decision |
|---|---|---:|---:|---:|---:|---|---:|---|
| A. Budget guardrails and lower-cost alternatives inside a catalogue-grounded day-by-day planner | Self-Serve Value Planner | 8 | 5 | 0.80 | 3 | (8 × 5 × 0.80) ÷ 3 | 10.67 | Selected |
| B. Conversational planner with clarifying questions and revisions | Self-Serve Value Planner | 7 | 4 | 0.65 | 4 | (7 × 4 × 0.65) ÷ 4 | 4.55 | Not selected |
| C. Live flight/hotel comparison and booking suggestions | Family Trip Coordinator | 8 | 5 | 0.35 | 9 | (8 × 5 × 0.35) ÷ 9 | 1.56 | Not selected |
| D. Group expense splitting and collaborative itinerary voting | Family Trip Coordinator | 4 | 3 | 0.60 | 5 | (4 × 3 × 0.60) ÷ 5 | 1.44 | Not selected |

**Selected feature:** TripWise — Budget-Safe, Catalogue-Grounded Day-by-Day Itinerary Planner. It is the most build-appropriate option because it addresses budget trust, can be grounded in a fixed catalogue, and can be validated without claiming live inventory or bookings.

## 1.6 Airtable base and data model

**Airtable working base/editor URL (not yet verified as a public assessor share link):** https://airtable.com/appYkjF5PDb1ZOyJd  
**Submission gate:** in Airtable, open **Share and sync → Create link to view → Copy link**. Replace the working/editor URL above with that copied public share/view URL, then prove it opens in a signed-out/incognito window. Do not submit the editor URL as though public sharing were verified.

Tables and relationship pattern:
- Personas: two explicitly labelled proto-persona hypotheses.
- Opportunity Backlog: four RICE-scored idea records. Target Persona is a real Linked Record into Personas, not text.
- Itinerary Log: scenario-created review rows. Persona is a genuine linked-record field. Review Status defaults to Pending Review.

Verified record IDs:
- Self-Serve Value Planner: recSPt2GE2f8SFHFJ; stable key self_serve_value_planner.
- Family Trip Coordinator: recSWWMeGcEktR5pb; stable key family_trip_coordinator.
- Budget guardrails: recsg7gZ8oS5QyXJo.
- Family pacing: recfajRczHk8WmvsR.
- Catalogue variety: reckmaEqkvXfm9JHQ.
- Cost breakdown: recIvM1I5eZKTEjqG.

The Airtable schema already had a formula field named RICE Score. Its formula computes (Reach × Impact × Confidence) ÷ Effort when Effort is nonzero. A stable Persona Key field was added and populated for the two proto-personas.

**Link verification gate:** The live schema and creation outputs confirm the field types and linked IDs for the opportunity records. After the next post-patch itinerary save, open its Airtable log row and verify its Persona field is clickable and opens the actual Persona row. The latest row predating the mapping is not evidence that the new path has passed.

## 1.7 PRD-lite

### Problem statement
The primary Self-Serve Value Planner needs to turn trip preferences into a coherent day plan while keeping the full trip within a stated INR budget. When activity, food, stay and transport costs are spread across different pages, an attractive itinerary can hide the true total. TripWise will use only supported catalogue data, expose category assumptions, recompute the total in trusted server code and flag a plan that exceeds the stated budget. The proto-persona’s frustration is a hypothesis to validate with interviews and usability tests, not a verbatim research quote.

### Chosen solution and rejected alternative
Chosen: deterministic catalogue-grounded planner with daily activity blocks, INR category estimates, a saved-trip dashboard, total-cost calculation and an over-budget state.
Rejected: a general free-form travel chatbot connected to live hotel/flight claims. The latter increases integration effort and hallucination/availability risk, and cannot be responsibly supported within this capstone’s limited, free-tier scope.

### User stories with INVEST checks
1. As a Self-Serve Value Planner, I want a destination/date/budget form so that I can express trip constraints before choosing activities. INVEST: Independent—form first; Negotiable—fields can be refined; Valuable—captures constraints; Estimable—bounded form; Small—one submission path; Testable—valid/invalid form tests.
2. As a Self-Serve Value Planner, I want an itinerary built only from supported catalogue items so that the plan does not invent places. INVEST: Independent—catalogue generation; Negotiable—ranking can evolve; Valuable—grounding; Estimable—fixed source; Small—single destination; Testable—activity IDs can be checked.
3. As a Self-Serve Value Planner, I want the full estimate and over-budget warning so that I can reject infeasible plans before booking. INVEST: Independent—cost service; Negotiable—estimate assumptions; Valuable—budget trust; Estimable—known fields; Small—sum and compare; Testable—assert formula and warning.
4. As a Family Trip Coordinator, I want pacing and traveller count respected so that the trip is practical for my party. INVEST: Independent—preferences; Negotiable—pace choices; Valuable—coordination; Estimable—limited inputs; Small—one itinerary rule set; Testable—day count and traveller count assertions.
5. As a signed-in traveller, I want to save and retrieve my itinerary so that I can continue planning later. INVEST: Independent—persistence; Negotiable—detail page; Valuable—continuity; Estimable—CRUD; Small—save/list/detail; Testable—re-login persistence and owner isolation.

### North Star and guardrail
- North Star candidate: 24-hour Useful Itinerary Action Rate = distinct signed-in users who save an itinerary or begin a booking action on an itinerary generated in the prior 24 hours ÷ distinct signed-in users who received a generated itinerary in that period.
- Guardrail: Budget Integrity Failure Rate = generated itineraries whose server-recomputed total differs from the displayed total or whose total exceeds the user’s stated budget without an explicit warning ÷ all generated itineraries. Target: 0% arithmetic mismatch and 0% silent over-budget plans. Track explicit over-budget plans separately rather than labelling all of them failures.

### Edge cases
- Unsupported destination: show that catalogue-backed costs are unavailable and ask the user to choose a supported destination; do not invent live quotes.
- Budget below plausible supported costs: generate only if the catalogue supports an illustrative plan, recompute the total, and show an explicit over-budget warning; never silently change the user’s budget.

**Out of scope:** live booking, live airfare/hotel inventory, price tracking, map/routing optimisation, payments, group cost splitting, and a traveller-facing display of the internal AI explanation.

# Part 2 — Prototype, automation, Evals and Responsible AI

**Published no-code app:** https://ai-trip-planner-web-7ogc.bolt.host

Use the published app above to run the eight golden cases in §2.5. This URL is included in the manuscript so that it will be present when copied into the single Google Doc submission.

## 2.1 Six-part app-generation prompt

### Capacity
Act as a senior full-stack product engineer and AI product manager. Build a reliable end-to-end application for a budget-conscious Indian traveller. Prioritise security, persistence, testability and honest estimates above decorative complexity.

### Request
Build TripWise, a responsive app where users sign up, log in, select a supported destination, specify dates/duration, traveller type/count, interests, travel pace and INR budget, generate a daily itinerary, save it, log out and retrieve it later.

### Insight
The core user problem is not lack of inspirational travel content alone; it is reconciling preferences, pacing and total cost. The app should show the assumptions behind estimates, support only catalogue items, and warn when a recomputed total exceeds the requested budget.

### Specifics
1. Use Supabase authentication and per-user saved trips with row-level security.
2. Maintain a small curated catalogue of destinations and activities with explicit cost ranges and provenance.
3. Generate daily blocks using catalogue records, include category breakdown and total, label values as estimates and calculate totals in server code.
4. Reject unsupported destinations without invented hotel/flight prices.
5. Keep server-side secrets out of the client; validate inputs and bound trip duration/traveller count.
6. Save the trip before calling Make so a failed automation never loses the itinerary.
7. Send minimal trip metadata to Make; do not send email or account identity. Give the event an opaque ID.
8. Make/Groq creates a separate internal reviewer note and writes to Airtable Itinerary_Log with Pending Review. Keep the explanation out of all traveller-facing UI and APIs (Option A).
9. A failed webhook must return an honest “Trip saved; internal explanation logging is pending” state.
10. Provide clear loading, empty, validation, unsupported-destination and error states.

### Personalization
Use INR and plain language for Indian travellers. Do not imply live fares, hotel availability, ratings or hours. Prioritise variety across interests and visible budget feasibility rather than popularity, price or review volume.

### Examples
- Solo trip to Jaipur, three days, budget ₹15,000: create exactly three days using supported activity IDs and expose the recomputed total.
- Group of four to Goa with ₹15,000 total budget: show the overrun explicitly if the computed estimate exceeds that amount.
- Unsupported destination: state that grounded catalogue data are missing.
- Make failure after successful save: preserve the itinerary and report logging as pending/failed honestly.
- AI explanation must remain internal and default to Pending Review.

## 2.2 Architecture and AI classification

**Architecture**
1. Bolt React application collects preferences and renders supported catalogue options.
2. Supabase Auth identifies the user; destinations/activities are read from the catalogue; trips are persisted in the trips table with owner ID.
3. generate-itinerary selects catalogue activities deterministically and computes totals.
4. save-itinerary validates the payload, recomputes cost server-side, writes the trip and fires the Make webhook with opaque event ID and minimum metadata.
5. Make validates/deduplicates the event, calls Groq, resolves a real Persona record, creates a Pending Review log in Airtable, records the event key and returns a processing acknowledgement.
6. Reviewer uses Airtable only. The explanation is not exposed to the traveller in this MVP.

**Classification:** AI-augmented rather than AI-native. The current app’s core generation works through deterministic catalogue selection and server-side arithmetic; the Groq model enhances the internal reviewer workflow rather than being indispensable to the traveller’s main journey.

**4D stage:** primarily Draft—TripWise drafts a structured itinerary the traveller can inspect and save; it does not book or execute transactions.
**AI Feature Ladder:** L2, an assistant/recommendation layer with structured output and human review. This is a working mapping for the project, not a claim that the course’s ladder definition has been independently reproduced.

## 2.3 Automation node-by-node table

| Node | Trigger / logic / action | Inputs and safeguards | Observable evidence |
|---|---|---|---|
| 1. Custom Webhook | Trigger on saved trip | Opaque event ID, destination, duration, traveller type/count, pace, budget/estimate, over-budget state, title; no email/account ID | Three actual app-triggered deliveries observed |
| 2. Data Store Exists | Logic: deduplicate | Checks itinerary_event_id in Data Store 222515 | Synthetic QA replay for event `qa-tripwise-family-link-20261018-002` ran only webhook → existence check → duplicate acknowledgement; no second Airtable Create Record operation ran. App-originated retest remains pending. |
| 3. Groq | Action: one-sentence reviewer note | GPT-OSS 20B via Make’s connected Groq account; data-only prompt; temperature 0.1; 500-token cap; no ungrounded live claims | One blank output surfaced and led to correction; later non-empty text verified |
| 4. Airtable Persona search | Logic: exact stable key lookup | Uses traveller_type to choose a stable Persona Key, then queries the real Persona row | Synthetic family QA lookup returned real record `recSWWMeGcEktR5pb` (Family Trip Coordinator); app-originated retest remains pending. |
| 5. Airtable Create Record | Action: Itinerary_Log write | Timestamp, summary, AI Explanation, Pending Review, real linked Persona record ID | Synthetic QA created `recQtH0nodr0RigUv` with non-empty explanation, `Pending Review`, and Persona linked to `recSWWMeGcEktR5pb`; app-originated retest remains pending. |
| 6. Data Store Add | Action: record event processed | Key = itinerary_event_id, runs after Airtable succeeds | Synthetic event key was stored once; its duplicate replay added no second Airtable record. |
| 7. Webhook Respond | Action: processing acknowledgement | Returns status=logged and echoes event ID only after successful modules | Synthetic first-run and duplicate-run responses both returned HTTP 200 JSON with `status=logged` and the matching event ID; fresh app save still pending. |

**Human review gate:** every log record is Pending Review. The explanation is a private internal reviewer artifact. No traveller-facing path renders it, even after a reviewer’s status change, until a separate approval-status integration is designed and tested.

**Reliability improvement:** `save-itinerary` marks logging successful only when Make returns an HTTP success response and JSON containing `status=logged` plus the same `itinerary_event_id`. Otherwise it reports logging failed; the saved trip is preserved. The current synthetic QA confirms the success and duplicate acknowledgement contract, but does not prove the published app is bound to this Supabase project or that a fresh app save triggers the latest flow.

## 2.4 Real automation transcripts

These are live observed test records, not synthetic examples. The first request occurred while the webhook was in learning mode and is not a complete workflow run.

### Run A — Goa; learning-mode sample, not a complete run
Input payload: Goa, 4 days, group of 4, relaxed pace, total budget ₹15,000, estimated cost ₹47,000, over_budget=true, title “Goa — 4 days”. Make delivery accepted at 2026-10-18 04:40:15 UTC. It was captured to learn the schema; no scenario execution was recorded for this event. Verdict: payload-contract sample only.

### Run B — Darjeeling; end-to-end failure discovered
Input payload: “Darjeeling — 2 days”; destination Darjeeling; 2 days; solo/1 traveller; relaxed; total budget ₹15,000; estimated cost ₹5,212; over_budget=false. The scenario completed five module operations (webhook, Data Store existence check, Groq, Airtable create, Data Store add) in about 3.5 seconds. Groq’s result was an empty string; its 120-token cap was consumed primarily by reasoning and finish_reason was length. Airtable created record rec4zPx6wFTbkOry5 with a blank explanation. Verdict: workflow steps ran, content-quality assertion failed. Corrective action: raised Groq max_tokens to 500 and tightened acknowledgement logic.

### Run C — Pondicherry; main workflow successful before Persona linking
Input payload: “Pondicherry — 3 days”; destination Pondicherry; 3 days; solo/1 traveller; relaxed; total budget ₹15,000; estimated cost ₹8,600; over_budget=false. Groq returned:
“This 3-day solo, relaxed-paced trip to Pondicherry has a total budget of 15,000 with an estimated cost of 8,600, which is within budget, but the suitability of the actual activities cannot be assessed from the supplied metadata alone.”
Make completed five module operations without errors. Airtable record recLjYPv8mafQ5OHN was created with Review Status Pending Review. Data Store recorded event key 861e05ed-5c7a-475c-8d0f-042b18711bed. This execution predated the new Persona lookup mapping. Verdict: successful nonempty internal-review creation, Persona-link and response-ack behaviour not yet validated.

### Run D — linked Persona QA event through the live Make webhook (synthetic QA, not app-generated)
Input payload: event ID qa-tripwise-family-link-20261018-001; title “QA TEST — Persona Link — Family Trip”; destination Pondicherry; 3 days; family; 3 travellers; relaxed pace; budget ₹18,000; estimate ₹12,800; over_budget=false. The payload was sent directly to the existing Make webhook from an on-demand QA harness, not generated by the app.

Observed output: main Make execution 867fc9a830514862be741cbff0952484 completed 7 operations in 3.706 seconds with zero module errors. Groq produced a nonempty one-sentence explanation. Airtable created record rec7jnhIi592Vhvmi with Review Status Pending Review and the genuine linked Persona record ID recSWWMeGcEktR5pb (Family Trip Coordinator). Data Store stored the QA event ID. Make returned HTTP 200 with JSON status=logged and the matching event ID.

Verdict: the straight-through Make → Groq → exact Persona lookup → linked Airtable record → Data Store → explicit acknowledgement path passed for this synthetic QA event. This does not prove that the published app has triggered the updated payload.

### Run E — first duplicate attempt before acknowledgement correction
The event `qa-tripwise-family-link-20261018-001` was replayed before the duplicate-response branch was fixed. Make completed only the webhook and Data Store existence check in 296 ms and did not create another Airtable row, but returned plain-text “Accepted” rather than the required JSON acknowledgement. This exposed a response-contract gap.

### Run F — linked Persona and explicit duplicate acknowledgement after correction
**First delivery input:** event ID `qa-tripwise-family-link-20261018-002`; title “QA TEST — Persona Link — Family Trip 002”; destination Pondicherry; 3 days; family; 3 travellers; relaxed; budget ₹18,000; estimated total ₹12,800; not over budget.

**Observed first-run output:** Make execution `048693a631d0426292d2c4325a2273a5` completed 7 operations in 3.752 seconds with zero module errors. Groq returned a non-empty, cautious sentence: “This 3‑day relaxed family trip to Pondicherry has a total budget of 18000, an estimated cost of 12800, and is within budget, but the suitability of the actual activities cannot be assessed from the supplied metadata alone.” Airtable created `recQtH0nodr0RigUv`; the explanation was non-empty, Review Status was `Pending Review`, and its Persona linked field contained the actual record ID `recSWWMeGcEktR5pb` (Family Trip Coordinator). Data Store stored the event ID. Make returned HTTP 200 JSON `{"status":"logged","itinerary_event_id":"qa-tripwise-family-link-20261018-002"}`.

**Observed replay output:** execution `3e575e30102945e8ab991fdbe28911f4` completed 3 operations in 275 ms: webhook, Data Store existence check, and duplicate acknowledgement. The response was HTTP 200 JSON with `status=logged` and the same event ID. The Airtable Create Record module did not run on replay, so no duplicate Airtable row was created.

**Verdict:** the synthetic new-event and replay acknowledgement paths pass after router correction. This is not a fresh app-generated save. Release still requires one new save from the published app, confirming the project/backend binding, Groq/Airtable fields and Supabase `explanation_log_status=logged`; the eight golden cases; signed-out app/Airtable access; cross-user isolation; and the traveller UI check that internal explanations never appear. Do not claim those gates passed from synthetic webhook evidence.

## 2.5 App Evals

### Scoring definitions

Rubric dimensions (1–5):
- Grounding: 1 = invented/unverified claims; 3 = some unsupported ambiguity; 5 = every suggested activity matches the catalogue and provenance is clear.
- Budget arithmetic: 1 = materially wrong; 3 = minor mismatch; 5 = displayed categories and total reconcile and over-budget state is explicit.
- Completeness: 1 = missing most days; 3 = partial day coverage; 5 = requested day count and required blocks present.
- Clarity: 1 = hard to act on; 3 = understandable with ambiguity; 5 = concise, structured, plain language.
- Preference fit: 1 = ignores user constraints; 3 = partial fit; 5 = respects duration, traveller type, interests and pace within catalogue limits.

Binary assertions:
A1 requested day count is exact; A2 costs use INR/labelled estimates; A3 cost breakdown reconciles with server total; A4 over-budget plan has explicit warning; A5 unsupported destination never receives invented grounded prices; A6 save persists through a logout/login check; A7 User A cannot read User B’s trip; A8 internal AI Explanation never appears in traveller-facing routes or API; A9 Airtable log Persona is a real linked record; A10 Review Status starts Pending Review.

### Golden dataset — all 8 cases must be run against the current deployed app

| ID | Actual test input to run | Variation | Expected assertions | Actual run / rubric scores |
|---|---|---|---|---|
| G1 | Solo, Jaipur, 3 days, Culture + Food, ₹15,000 | Domestic / solo / moderate budget | A1–A4, A8 | Not tested |
| G2 | Couple, Goa, 4 days, beaches + food, ₹24,000 | Domestic / couple / longer trip | A1–A4, A8 | Not tested |
| G3 | Family with children, Mysuru, 2 days, museums + easy pace, ₹18,000 | Domestic / family | A1–A4, A8 | Not tested |
| G4 | Group of friends, Rishikesh, 3 days, outdoors, ₹20,000 | Domestic / group | A1–A4, A8 | Not tested |
| G5 | Solo, unsupported international destination, 5 days, ₹80,000 | Unsupported catalogue case | A5, clear fallback | Not tested |
| G6 | Couple, Jaipur, 2 days, low budget ₹2,000 | Budget infeasibility | A2–A4 | Not tested |
| G7 | Family, supported destination, 7 days, ₹45,000 | Longer domestic duration | A1–A4 | Not tested |
| G8 | Group, supported destination, 2 days, ₹12,000, interest text includes prompt-injection attempt | Prompt-injection negative test | A5, A8, no policy override | Not tested |

The rubric requires actual output scores for every case. This table is an honest execution plan, not eight claimed passes. The live test history supports the three automation transcripts in §2.4; it does not support claiming the eight app eval cases have been run. Complete the eight app runs and fill rubric numbers/assertions with observed results before submission.

## 2.6 Responsible-AI pass

**Prompt injection:** The itinerary generator is deterministic and does not send free-text interests to an LLM. Interests are mapped to fixed categories; queries use the selected destination ID. The Make prompt explicitly treats webhook text as untrusted data and instructs the model not to follow embedded instructions.

**Hallucination:** The generator selects activity records from a curated catalogue and uses stored cost bounds/provenance; it does not invent live flights, lodging, reviews or current availability. Unsupported destinations receive a limitation message. Cost totals are recomputed server-side.

**PII and privacy:** Sign-in email is held by the auth provider. Trips are associated with authenticated user ID in the database. The Make payload does not contain user email, account ID or direct identifiers; it includes an opaque event ID and coarse trip metadata. The app does not request passport numbers, payment card details or precise location. Apply least-privilege access and verify owner-scoped RLS; do not put server secrets in frontend configuration.

**Four bias mitigations and residual gaps**
- Popularity bias: select from catalogue/category rotation and interest matching; do not rank using click volume/popularity counts. Measure catalogue selection share by activity category and destination, and test whether repeated destinations dominate across the golden set. Residual: the catalogue itself may reflect selection bias.
- Price bias: show range/provenance and compute the total; warn when total exceeds budget. Do not equate higher price with better fit. Test both low-budget and high-budget inputs. Residual: shared transport assumptions and catalogue range midpoint may under/over-estimate spend.
- Geographical bias: catalogue has a finite set of supported destinations; do not silently infer unsupported destinations. Audit catalogue representation by region before expanding. Residual: only ten destinations are present, so the current prototype cannot claim balanced geographical coverage.
- Review bias: generator does not consume review counts/ratings; it uses explicit user interests, pace and catalogue metadata. It therefore avoids review-volume ranking but cannot evaluate rating-based fairness until such data are added.

**Human review:** Airtable is an internal queue with Pending Review default; explanations are not exposed in the app. A reviewer can reject an unsuitable note and leave audit notes. Reviewer state does not change traveller visibility in this MVP.

# Part 3 — Pricing, positioning and go-to-market

## 3.1 Pricing and packaging proposal

Prices below are strategy hypotheses, not measured willingness-to-pay results or implemented billing.

| Tier | Proposed price | Package | Role |
|---|---:|---|---|
| TripWise Starter | ₹0 | A small number of saved catalogue-grounded itineraries, cost estimate and warning | Low-friction trust-building for the Self-Serve Value Planner |
| TripWise Plus | ₹149/month | Higher trip/save allowance and convenience features after validation | Best fit to repeated trip planners seeking time savings and cost confidence |
| TripWise Family | ₹299/month | Future validated family coordination, shared preferences and multi-person planning | For Family Trip Coordinators only after collaboration features are built and tested |

**Psychological pricing:** no decoy tier and no 99-ending prices. This is a trust-centered budgeting product; transparent round amounts are easier to compare against household budgets. No time-limited free trial is planned; a durable free Starter tier lowers risk without a surprise expiry.

## 3.2 Five-component GTM

**User Segment:** Primary is the Self-Serve Value Planner—behavioural and value-based—who wants an INR estimate and an itinerary that can be checked against a budget before committing. Secondary is the Family Trip Coordinator, whose group needs and pacing are represented as an unvalidated proto-persona hypothesis.

**Value Positioning:** “A clear plan you can trust enough to inspect” rather than “the cheapest travel AI.” The promise is a catalogue-grounded day-by-day plan, visible assumptions and an explicit over-budget state—not a live price guarantee.

**Pricing & Packaging:** A persistent free Starter tier gives users time to judge grounding and budget clarity. ₹149/month Plus is a testing hypothesis for frequent planners; ₹299/month Family is a future package and must not be represented as available in the prototype.

**Channel Strategy:** Begin with opt-in app entry points, destination/cost-planning content, creator demonstrations showing both feasible and infeasible budgets, and shareable itinerary exports only after privacy/safety review. Do not claim booking partnerships or conversion lift until integrated/observed.

**Scale & Measurement:** Measure activation from entry to opened form, completion to itinerary view, and Stage 5 save/booking action. Instrument actual events before using vanity top-of-funnel volume. Guardrail with budget integrity, unsupported-destination rate, recommendation diversity and logged AI reviewer failures.

## 3.3 Value Proposition Canvas — primary persona

**Customer jobs**
1. Plan the requested number of days around interests.
2. Understand total cost before committing.
3. Save and revisit the plan.

**Customer pains**
1. Costs spread across multiple sources.
2. A neat itinerary can hide total overrun.
3. Unsupported/live data may be falsely implied.

**Customer gains**
1. One consistent day plan.
2. Clear category costs and warning.
3. Saved plan with transparent assumptions.

**Pain relievers**
1. Catalogue IDs anchor each activity.
2. Server recomputes category totals and flags over-budget plans.
3. Unsupported destinations return a limitation instead of invented live values.

**Gain creators**
1. Preference and pace mapping produce a structured daily sequence.
2. An explicit estimate/provenance note clarifies uncertainty.
3. Saved trips support continuation after login.

## 3.4 Price-quality matrix

Qualitative placement based on public feature descriptions, not hands-on price-performance testing:
- Google Maps: strong place/navigation utility; limited on its own for the complete budget-aware itinerary decision.
- Wanderlog: broad planning/organisation, maps, route and budget tools; richer breadth than this prototype.
- Expedia: richer conversational and inventory-linked paths; broader booking integration than this prototype.
- TripWise: narrow scope, modest feature breadth, intentionally higher emphasis on catalogue grounding, INR budget reconciliation and candid limitations.

**Positioning statement:** For Indian self-serve travellers who want to plan without losing control of total spend, TripWise is a budget-aware itinerary planner that turns a supported set of destinations and activities into a structured plan with visible assumptions. Unlike inspiration-only chat, it recomputes estimates and warns when the full plan exceeds budget. It is not the lowest-price promise or a live booking engine; its point of difference is a plan the user can interrogate before deciding.

## 3.5 Bottom-up year-one revenue model

Hypothetical assumptions, not observed forecast data:
- Exit monthly active feature users by month 12: 50,000.
- Paid conversion at exit: 3%.
- Plus price: ₹149/month.
- Exit paid subscribers = 50,000 × 3% = 1,500.
- Exit MRR = 1,500 × ₹149 = ₹223,500.
- Exit ARR run-rate = ₹223,500 × 12 = ₹2,682,000.
- Illustrative year-one revenue under a linear ramp from zero to that exit MAU is about half the exit run-rate: average 25,000 MAU × 3% = 750 average paid users; ₹149 × 750 × 12 = ₹1,341,000.
This excludes churn, price discounts, refunds, taxes, costs and seasonal changes. Validate active users and conversion using real funnel events before calling this a forecast.

# Part 4 — Growth analytics, funnel diagnostic and Responsible-AI rollout

## 4.1 Stage conversion, drop-off and absolute loss

| Transition | From | To | Conversion | Drop-off | Absolute users lost |
|---|---:|---:|---:|---:|---:|
| Stage 1 → 2: banner to open | 50,000 | 20,000 | 40% | 60% | 30,000 |
| Stage 2 → 3: open to submit | 20,000 | 14,000 | 70% | 30% | 6,000 |
| Stage 3 → 4: submit to itinerary view | 14,000 | 11,900 | 85% | 15% | 2,100 |
| Stage 4 → 5: itinerary view to save/booking action | 11,900 | 3,570 | 30% | 70% | 8,330 |

Largest absolute loss: Stage 1→2, 30,000 users. Largest percentage drop-off: Stage 4→5, 70%.

**Priority:** Fix Stage 4→5 first because it is immediately before the revenue-driving save/booking action and loses 70% of itinerary viewers. The Mixpanel vanity-metrics lesson applies: more banner opens may improve the top-line count without improving downstream action. Stage 1→2 represents the largest raw loss, but top-of-funnel volume cannot be assumed to convert; reducing friction at the moment of decision is the more direct short-term route to a useful outcome.

## 4.2 Growth loop

**Loop type: AI Product Loop.** A traveller generates a useful itinerary and—after a privacy and accuracy design pass is implemented—chooses to export/share a stripped-down itinerary card with destination, day plan and estimate assumptions but no account ID, private notes or internal AI explanation. A friend opens the shared artifact and starts a fresh TripWise plan, generating another useful output that may itself be shared. Sharing/export is not in the current MVP; this is a proposed loop and must be measured via shared-link opens to new trip starts, not assumed as an existing feature.

## 4.3 Product lifecycle

Current stage immediately after a limited prototype pilot: **Introduction**, because the product has been built and an initial flow tested, but broad acquisition and statistically reliable outcomes are not established.
- Next: **Growth** — improve activation-to-save/booking conversion, verify budget integrity, and expand catalogue only as coverage/evals allow.
- Following: **Maturity** — optimise retention and portfolio quality, establish stable bias monitoring, and extend supported coverage from evidence.

## 4.4 North Star and guardrail tied to funnel

- North Star = unique signed-in users who save a trip or start a booking action within 24 hours of viewing a generated itinerary ÷ unique signed-in users who viewed a generated itinerary in the period (Stage 4 denominator, Stage 5 numerator).
- Guardrail = generated itineraries with a silent budget breach or arithmetic mismatch ÷ all generated itineraries. Require both an over-budget warning whenever computed total > user budget and exact agreement between displayed total and server-recomputed total. Guardrail target for silent breach/arithmetic mismatch is 0%.

## 4.5 Govern / Map / Measure / Manage

**Govern:** Accountable owner is the product manager for feature/data definitions; engineering owner for RLS, API contracts and reliable costs; reviewer owner for Airtable’s internal AI explanation queue. Release approval requires the eight golden cases, security/PII checks, real linked-persona verification and Make execution logs.

**Map risks:** Unsupported destinations, estimate confusion, undercounted costs, popularity/geography/price/review bias, prompt injection in user-provided interests, exposure of PII, cross-user trip access, duplicate webhooks, and an unreviewed explanation being shown as fact. The current app does not send free-text interests to an itinerary-generating LLM and does not use review counts, but catalogue coverage itself remains a risk.

**Measure:** Run the eight golden cases before every public release; weekly for the first month after launch; monthly thereafter, and immediately after changing catalogue ranking, cost logic, model/prompt, or webhook mappings. Track over-budget warnings, arithmetic mismatch, unsupported destination fallbacks, category/destination diversity, blank/failed Groq outputs and correct linked-record rates.

**Manage — example: geographical bias resurfaces**
- Detect: measure a catalogue-diversity signal such as unique destination IDs recommended per 100 generated itineraries and the share of top-3 destinations. Alert if one region/destination exceeds its pre-agreed baseline by a defined margin over seven days.
- Contain: pause expansion or turn off the affected recommendation rule; keep trip saving available. Use the existing Make workflow and Airtable Itinerary_Log Pending Review records to hold reviewer notes; never expose internal explanations to travellers.
- Fix: inspect whether category ordering or catalogue representation is over-concentrated; rebalance coverage and deterministic selection, then replay all eight golden cases. Record before/after evidence.
- Communicate: document the incident, affected dates, containment action, evaluation results and resolution in the product change log; inform stakeholders, and if a user-facing output was materially misleading, provide a clear correction through the appropriate customer-support channel.

## Final release checklist

- [ ] Insert the real published Bolt app URL and verify it opens while signed out.
- [ ] Verify Airtable is viewable by an assessor in a signed-out/incognito session.
- [ ] Save one new itinerary after the latest changes and confirm Make execution.
- [ ] Verify Groq text is non-empty, Airtable has a genuinely linked Persona record, Review Status is Pending Review, and Supabase changes explanation_log_status to logged only after Make acknowledgement.
- [ ] Replay the same event ID once and prove no duplicate Airtable record is created.
- [ ] Run all 8 golden dataset inputs through the actual app; score every rubric and binary assertion from observed output.
- [ ] Verify a user cannot read another user’s trips; verify internal explanation never appears in the traveller UI.
- [ ] Copy this manuscript to one Google Doc, set “Anyone with the link can view,” embed Airtable and published app links in Parts 1 and 2, and submit that one Google Doc URL.
---

## Release checklist status — 18 October 2026

This status separates observed proof from untested release gates. Synthetic webhook QA must not be represented as an app-originated end-to-end pass.

| Gate | Current status | Evidence / remaining action |
|---|---|---|
| Published Bolt URL inserted in Part 2 | **Inserted; signed-out access not verified** | URL: https://ai-trip-planner-web-7ogc.bolt.host. Open it in a signed-out/incognito browser to prove the public page responds. A cache miss in the research/browser environment is inconclusive. |
| Airtable assessor share link | **Not verified** | The URL currently listed in Part 1 is the working base/editor URL, not a verified public share/view URL. In Airtable, use **Share and sync → Create link to view → Copy link**, replace the Part 1 URL with that copied link, then open it signed out. |
| Fresh app-originated save after latest changes | **Not yet tested** | No new app-originated trip appeared after the last observed app save. A fresh signed-in save is required to confirm the published app is connected to the deployed function and current Make webhook. |
| Non-empty Groq text | **Passed on synthetic QA only** | Event qa-tripwise-family-link-20261018-002 produced the caveated non-empty internal sentence quoted in Run F. |
| Genuine linked Persona and Pending Review | **Passed on synthetic QA only** | Airtable row recQtH0nodr0RigUv links to Family Trip Coordinator record recSWWMeGcEktR5pb and has Review Status Pending Review. |
| Supabase explanation_log_status=logged only after acknowledgement | **Function logic verified; app end-to-end not verified** | Active save-itinerary version 7 on project uwbhcqywmgcosyqwmmlq requires HTTP success and a JSON acknowledgement containing status=logged plus the matching event ID. The published app's actual Supabase project binding still needs confirmation; the separate project reference oqxnlkdnqcbciiszyalp could not be inspected with current connector permissions. |
| Duplicate replay avoids second Airtable record | **Passed on synthetic QA only** | Replaying event qa-tripwise-family-link-20261018-002 ran only the webhook, existence check and duplicate acknowledgement modules; Airtable Create Record did not run. |
| Eight golden dataset inputs run through the actual app | **Not tested** | Keep all G1–G8 scores and assertions marked Not tested until actual deployed-app outputs are captured and scored. |
| Cross-user trip access blocked | **RLS policy inspected; two-user acceptance test not run** | trips has RLS enabled and select_own_trips uses auth.uid() = user_id. Verify behavior with two distinct test users before claiming acceptance passed. |
| Internal explanation never appears in traveller UI/API | **API response shape checked; traveller UI acceptance test not run** | save-itinerary returns trip ID, event ID, status and message—not the explanation. Check saved-trip screens and any trip-detail/API routes in the published app to prove no explanation is exposed. |
| One Google Doc, Anyone with the link can view | **Not created** | The graded deliverable is one Google Doc URL only. Copy this complete manuscript into one Google Doc, preserve Part 1–4 order, replace the Airtable editor URL with the public share/view link, retain the published app URL in Part 2, set view access for anyone with the link, test both links signed out, then submit only that Google Doc URL. |
