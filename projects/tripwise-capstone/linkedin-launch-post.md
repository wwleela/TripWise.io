I built TripWise as an end-to-end AI Product Management capstone around a simple question:

**How do we make AI-assisted trip planning useful without making it overconfident?**

The prototype takes a traveller’s destination, duration, interests, pace and INR budget, then generates a catalogue-grounded day-by-day itinerary, recomputes the estimated cost and flags over-budget plans.

I connected the app to Supabase for authentication and saved trips, then built a Make → Groq → Airtable workflow for an internal, human-review-only “Why this itinerary?” note. The review is kept out of the traveller-facing app.

The integration work taught me as much as the UI:
- A webhook being accepted is not proof that an automation completed.
- An AI response that technically returns successfully can still be blank.
- A typed persona name is not the same as a real linked record.
- “AI-powered” is not a product strategy; the model needs a bounded job, evidence and guardrails.

One live test exposed an empty Groq explanation due to its token cap. I corrected the output budget, reran the workflow and verified a non-empty explanation and Airtable record. A labelled webhook QA test now verifies the linked Persona write and the duplicate-event acknowledgement path. The remaining integration gate is one fresh save initiated inside the published app; I’m not presenting the eight-case app evaluation suite as complete until the cases are actually run.

My current product stance: optimise for budget integrity and user trust before adding live inventory, booking claims or feature breadth.

This is a working prototype, not a production travel recommendation system. My next release gate is to verify the linked Persona field and duplicate-event behavior, then score all eight golden cases across grounding, arithmetic, completeness, clarity and preference fit.

#AIProductManagement #ProductManagement #ResponsibleAI #TravelTech #NoCode #Supabase #Make #Groq #Airtable

Try the prototype: https://ai-trip-planner-web-7ogc.bolt.host

Portfolio case study: https://github.com/wwleela/TripWise.io/tree/main/projects/tripwise-capstone
