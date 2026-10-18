# TripWise eval dataset, rubric and run tracker

## Rubric (rate each output from 1 to 5)
| Criterion | 1 | 3 | 5 |
|---|---|---|---|
| Catalogue grounding | Invented or unsupported activity | Mixed grounding / ambiguity | Every activity maps to supported catalogue |
| Budget arithmetic | Materially wrong | Minor mismatch | Category totals reconcile with server total |
| Completeness | Most requested days absent | Partial coverage | Exact duration and required blocks |
| Clarity | Hard to use | Understandable with ambiguity | Clear, structured and concise |
| Preference fit | Ignores constraints | Partial fit | Duration/type/interests/pace respected within supported data |

## Binary assertions
A1 exact requested day count; A2 all costs labelled INR estimates; A3 displayed categories reconcile with server total; A4 over-budget output has warning; A5 unsupported destination receives no invented grounded price; A6 trip persists across logout/login; A7 one test user cannot read another user's trips; A8 AI Explanation never appears in traveller-facing UI/API; A9 created Airtable log has true linked Persona record; A10 Review Status defaults to Pending Review.

## Golden dataset and actual run status
Do not fill scores based on expectation. Run each input in the published app, preserve the actual output text, and score 5 rubric values plus the 10 assertions.

| ID | Input | Required diversity | Run status | Score and assertions |
|---|---|---|---|---|
| G1 | Solo; Jaipur; 3 days; culture + food; ₹15,000 | Domestic; solo | Not tested | Not tested |
| G2 | Couple; Goa; 4 days; beach + food; ₹24,000 | Domestic; couple | Not tested | Not tested |
| G3 | Family with children; Mysuru; 2 days; museums/easy pace; ₹18,000 | Domestic; family | Not tested | Not tested |
| G4 | Group of friends; Rishikesh; 3 days; outdoors; ₹20,000 | Domestic; group | Not tested | Not tested |
| G5 | Solo; unsupported international destination; 5 days; ₹80,000 | Unsupported / international | Not tested | Not tested |
| G6 | Couple; Jaipur; 2 days; ₹2,000 | Budget infeasibility | Not tested | Not tested |
| G7 | Family; supported destination; 7 days; ₹45,000 | Long duration / family | Not tested | Not tested |
| G8 | Group; supported destination; 2 days; ₹12,000; interest includes instruction-like prompt injection | Prompt injection | Not tested | Not tested |

## Existing automation evidence (not the eight app evals)
- Goa: a real webhook delivery was accepted while Make was in learning mode, but no workflow execution occurred. Do not count as end-to-end pass.
- Darjeeling: five modules executed, but Groq output was blank under a 120-token cap; Airtable record created blank. This is a failed content-quality test.
- Pondicherry: five modules executed, Groq returned a non-empty caveated sentence, Airtable record recLjYPv8mafQ5OHN created with Pending Review, and Data Store recorded the event. This predates the new linked Persona routing.

## Required test evidence per case
For each golden case paste: (1) exact input; (2) actual app output; (3) five rubric scores with one-line rationale; (4) A1-A10 pass/fail/NA with evidence; (5) any bug fix and rerun. If a check was not exercised, leave it Not tested rather than infer a pass.

