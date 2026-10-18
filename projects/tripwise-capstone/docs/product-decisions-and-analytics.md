# TripWise — Product decisions, analytics and release gates

## Decision log

| ID | Decision | Why | Trade-off / what would change the decision |
|---|---|---|---|
| D1 | Focus on a budget-aware catalogue-grounded itinerary rather than a general chat assistant | Budget accuracy and bounded grounding are evaluable within a small no-code prototype | Revisit only after user research shows follow-up conversation is the primary blocker |
| D2 | Keep itinerary generation deterministic | Catalogue grounding and cost recomputation are easier to audit than unconstrained itinerary generation | A model may help in a later version, but only behind schema validation and golden-set evaluation |
| D3 | Use Groq only for internal reviewer explanation | Keeps a human review checkpoint without presenting an unreviewed explanation as truth | Expose explanations to users only after an approval-state integration and usability/safety tests |
| D4 | Save the trip before calling Make | Automation failure must not erase user value | Add retry/reconciliation only with clear idempotency semantics |
| D5 | Use an opaque event ID and Data Store deduplication | Allows event-level correlation without sending account identity | Durable idempotency and retry semantics need a repeat-delivery test before public release |
| D6 | Require Make response to echo the event ID before marking logging as logged | A generic HTTP 200/accepted response is not proof that Groq and Airtable succeeded | Keep log status failed/pending on missing, malformed or mismatched acknowledgement |
| D7 | Map persona via deterministic Airtable linked record | A typed persona label is not a relational link; use stable key + actual record ID | If no exact record is found, fail safely and flag the integration rather than inventing a record |
| D8 | Keep live prices, bookings and international unsupported destinations out of scope | Avoid unverified inventory/price claims and scope creep | Add only after authoritative data sources and refresh/expiry behavior exist |
| D9 | Treat personas and RICE assumptions as hypotheses | No direct user-interview data has been supplied for the two proto-personas | Conduct interviews/task tests and update confidence based on observed evidence |
| D10 | Score all eight golden cases before calling the evaluation complete | Happy-path demos do not show failure coverage | Block launch claims until outputs and assertion evidence are recorded for each case |

## Analytics event taxonomy

These are instrumentation specifications unless a corresponding event is explicitly observed in live logs. Do not report a dashboard or event stream as live until implemented and checked.

| Event | When emitted | Required properties | Privacy constraints |
|---|---|---|---|
| trip_assistant_entry_viewed | Entry banner/view appears | timestamp, session-scoped pseudonymous ID, surface | No direct email/account identifier in analytics event |
| trip_assistant_opened | User opens planner | timestamp, acquisition surface | No precise location |
| trip_preferences_submitted | Valid preference form submitted | traveller_type, destination_id, duration_days, interests count, travel_pace, budget band | Avoid free-text interests in analytics |
| itinerary_generated | Itinerary generation returns | destination_id, duration_days, catalogue activity IDs/categories, status | No user email |
| itinerary_viewed | Result is rendered | destination_id, duration, estimated total band, over_budget | Use budget bands where exact amount isn't required |
| trip_save_succeeded | Saved trip write succeeds | opaque event ID, duration, over_budget | Event ID should be random/non-identifying |
| trip_save_failed | Save write fails | error category, no raw server error or secret | Never send tokens/SQL output |
| internal_review_webhook_sent | save-itinerary fires Make webhook | opaque event ID, status | No user email, account ID or starting-city free text |
| internal_review_generated | Groq produces non-empty text | event ID, model alias, latency, pass/fail | Do not send explanation text to analytics |
| internal_review_log_created | Airtable row created | event ID, review status, persona key | Reviewer text stays in Airtable queue |
| persona_link_resolution_failed | Exact key matches no record | persona key, error type | Do not create a persona based on model inference |
| internal_explanation_reviewed | Reviewer changes queue status | event ID, approved/rejected/needs-edit | No public exposure implied |

## Metric definitions

- Useful Itinerary Action Rate (24h) = unique signed-in users who save the itinerary or initiate a booking action within 24 hours after an itinerary view ÷ unique signed-in users who viewed a generated itinerary in that period.
- Budget Integrity Failure Rate = generated trips whose displayed total differs from server-recomputed total OR whose estimate exceeds user budget without a visible warning ÷ all generated trips.
- Grounded Activity Rate = selected itinerary time blocks whose activity_id exists in the supported activities table ÷ all selected itinerary time blocks.
- Unsupported Destination Fallback Rate = unsupported-destination requests with an explicit limitation response ÷ all unsupported-destination requests.
- Persona Link Success Rate = Airtable review rows with a valid linked Persona record ÷ all Airtable review rows created.
- Nonempty Review Explanation Rate = review rows with nonempty, schema-compliant AI Explanation ÷ all review rows created.
- Duplicate Record Rate = duplicate Itinerary_Log records for one event ID ÷ distinct itinerary_event_id values delivered to Make.
- Review Gate Integrity = traveller-facing pages/APIs containing internal AI Explanation should be zero.

## Prioritised backlog and RICE assumptions

Reach uses a relative 1–10 score, Impact 1–5, Confidence 0–1 and Effort in team-weeks. RICE = Reach × Impact × Confidence ÷ Effort. The values below are prioritisation assumptions, not measured audience counts or statistically supported confidence.

| Idea | Reach | Impact | Confidence | Effort | RICE | Decision |
|---|---:|---:|---:|---:|---:|---|
| Budget guardrails and lower-cost alternatives | 8 | 5 | 0.80 | 3 | 10.67 | Selected |
| Family pacing and interest-fit controls | 6 | 5 | 0.70 | 4 | 5.25 | Selected in Airtable; revisit after prototype scope review |
| Catalogue variety beyond top popular destinations | 5 | 4 | 0.60 | 3 | 4.00 | Proposed |
| Assumption-transparent category cost breakdown | 7 | 4 | 0.65 | 4 | 4.55 | Proposed |

Note: the assignment expects a single Selected idea that matches the one chosen product feature. The Airtable backlog currently has the top budget-guardrail idea and family-pacing idea marked Selected in its records; align the final Airtable status to one Selected feature if the assessor interprets this as exactly one selected concept.

## Release readiness checklist

### Must pass before public capstone claim
- Public app URL loads and sign-in is functional.
- Saved trip persists across a real logout/login.
- RLS prevents cross-user access.
- Server-side estimate reconciles; over-budget warning is visible where needed.
- Unsupported destination behaves honestly.
- Fresh post-change app save is received by active Make scenario.
- Groq explanation is nonempty and bounded.
- Airtable review row is Pending Review and has a real clickable Persona link.
- Supabase log status only becomes logged after matching Make acknowledgement.
- Duplicate event replay does not create a second Airtable row.
- Eight golden cases are actually run and all rubric/assertion cells are populated.
- Airtable base is viewable by assessors without edit rights.
- Master manuscript is copied to the single required Google Doc and shared read-only.

### Known residual risks
- Only ten destinations and fifty-two activities are present; geographical fairness cannot be claimed.
- Prices are indicative and not live quotes.
- Generator is deterministic, not LLM-generated.
- The webhook sends trip metadata, not full activity sequence; the Groq note appropriately limits what it claims.
- Eight app evals, duplicate replay and new Persona-linked log after the latest changes are not yet verified.
