# TripWise pricing, GTM and growth plan

## Price and packaging hypothesis
| Tier | Price | Rationale |
|---|---:|---|
| Starter | ₹0 | Gives the Self-Serve Value Planner a low-friction way to test cost clarity and catalogue limits. |
| Plus | ₹149/month | Hypothesised willingness-to-pay for repeat travellers who value saved plans, convenience and budget clarity. |
| Family | ₹299/month | Future tier for validated collaborative family workflows; not implemented in the prototype. |

Psychological pricing: deliberately avoid decoy pricing and 99-ending prices. Clear round numbers align with the product's trust/budget positioning. Use a permanent free tier rather than a time-limited trial to avoid post-trial surprise. These are hypotheses; validate through interviews, price-choice tests and actual purchase behavior before implementing billing.

## GTM framework
- User Segment: Primary is Self-Serve Value Planner, a behavioural/value-based proto-persona. Secondary is Family Trip Coordinator. Neither has been validated through primary interviews yet.
- Value Positioning: budget-aware plans with transparent assumptions and a clear warning—not “the cheapest travel AI” and not a live quote engine.
- Pricing & Packaging: Starter ₹0, Plus ₹149/month, Family ₹299/month after group collaboration is built. Keep monetisation out of the capstone prototype.
- Channel Strategy: opt-in in-app placement, cost-planning articles and creator demos that show feasible and over-budget examples. Proposed future share/export is not built; keep it out of current capability claims.
- Scale & Measurement: instrument entry banner, feature open, preference submit, itinerary view, save/booking action; use the 24-hour useful itinerary action rate as the North Star and budget integrity as a guardrail.

## Value Proposition Canvas — primary Self-Serve Value Planner

Jobs:
1. Convert destination, dates and interests into a daily plan.
2. Know the full estimated spend before committing.
3. Save and revisit the plan.

Pains:
1. Costs fragmented across stay/food/transport/activities.
2. Hidden overruns or false confidence in estimates.
3. Unclear whether a suggested activity is a real catalogue option.

Gains:
1. Coherent days and less manual stitching.
2. Category totals and a visible over-budget state.
3. Persistent, retrievable trip.

Pain relievers:
1. Fixed activity IDs and provenance notes reduce unsupported recommendations.
2. Server-side cost recomputation plus over-budget warning exposes infeasibility.
3. Unsupported destinations receive an explicit limitation.

Gain creators:
1. Interest/category mapping makes the itinerary personal within catalogue limits.
2. Category cost breakdown shows why the total is what it is.
3. Authenticated save/list/detail flow maintains continuity.

## Price-quality matrix: qualitative, public-source based
- Google Maps: strong navigation and saved-list utility; not the complete budget-aware itinerary workflow.
- Wanderlog: high planning breadth (maps, routes, reservations, budgeting, collaboration and AI assistance).
- Expedia: conversational planning and closer connection to bookable inventory.
- TripWise: narrow breadth, strong emphasis on cost transparency, catalogue grounding and explicit limitations. This is a prototype position, not a proven market ranking.

Positioning statement: For Indian self-serve travellers who want a plan without losing control of total spend, TripWise turns supported destination/activity data into a structured daily plan with transparent assumptions. It does not promise the lowest price or live bookings; its intended difference is a plan the user can inspect and challenge before deciding.

## Bottom-up year-one revenue estimate
Assumptions (not measured):
- 50,000 monthly active feature users by month 12.
- 3% conversion to Plus = 1,500 paid subscribers at exit.
- Plus price ₹149/month.
- Exit MRR = 1,500 × ₹149 = ₹223,500.
- Exit ARR run-rate = ₹223,500 × 12 = ₹2,682,000.
- If MAU ramps linearly from zero to 50,000 during year one, average MAU = 25,000; average paid subscribers at 3% = 750; year-one revenue estimate = 750 × ₹149 × 12 = ₹1,341,000.
This excludes churn, discounts, refunds, tax, operating costs and seasonality. It is a scenario model, not actual revenue or a forecast from observed cohort data.

## Growth funnel math from the supplied pilot
| Transition | Conversion | Drop-off | Users lost |
|---|---:|---:|---:|
| Entry banner 50,000 → Open 20,000 | 40% | 60% | 30,000 |
| Open 20,000 → Submit 14,000 | 70% | 30% | 6,000 |
| Submit 14,000 → View 11,900 | 85% | 15% | 2,100 |
| View 11,900 → Save/booking 3,570 | 30% | 70% | 8,330 |

Largest absolute loss is Stage 1→2 at 30,000; largest percentage loss is Stage 4→5 at 70%. Prioritise the last step before the value/revenue action. As the Mixpanel vanity-metric lesson shows, adding top-of-funnel sign-ups or opens without improving downstream actions may not improve outcomes.

## Growth loop and lifecycle
Proposed AI Product Loop: a traveller generates a useful itinerary, then (once privacy-safe export is implemented) shares a stripped-down itinerary card; a recipient opens it, starts a new trip and generates a new itinerary that may be shared. This loop is not implemented in the MVP.

Lifecycle after pilot: Introduction. Next Growth focus: lift itinerary-view-to-save/booking conversion without worsening budget integrity. Following Maturity focus: improve retention, catalogue coverage and monitoring with stable cohort evidence.
