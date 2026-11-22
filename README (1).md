# ClearAir — Urban Air Quality Assistant

**PromptWars (Hack2skill) — Challenge 6: Urban Air Quality Assistant**

## Problem
Families in cities with severe winter smog need practical, personalized guidance — not generic air quality tips — to protect their health and reduce their own contribution to pollution.

## Approach
ClearAir uses a hybrid architecture rather than a single prompt wrapped around a form:

1. A rule engine first computes an AQI severity tier, commute exposure risk, indoor air risk, and priority flags (sensitive household members, outdoor workers, limited vehicle access) directly from the form inputs.
2. That computed assessment is handed to Claude (Anthropic) as ground truth. Claude's role is to explain it and turn it into a personalized, four-part plan — not to re-judge severity itself.
3. After the initial plan, the person can ask follow-up questions in a small chat; Claude answers using the full conversation and original assessment as context.

The plan covers five sections: today's low-emission routine, indoor air quality hacks, sustainable commute alternatives, **community actions** (carpools, reporting pollution sources, local air-quality groups — addressing *communities*, not just the individual household), and why it matters for this specific household.

The city field isn't cosmetic: a small lookup recognizes a few high-smog regions (Delhi/NCR, Lahore/Punjab, Dhaka) and adds the actual local pollution driver (crop-residue burning and inversion vs. brick-kiln/vehicle emissions) as a signal, so the plan is genuinely tailored to location, not just labeled with it.

## Access note
The live Claude call requires the viewer to be signed into a Claude.ai account (a platform requirement of this hosting, not an app limitation). So the app never depends on that: if Claude isn't available for any reason — signed out, permission not granted, rate-limited — it falls back instantly to the same rule engine's deterministic output, so every visitor sees a complete, working plan with no sign-in required and no dead end. Signed-in visitors who grant permission get the richer, live Claude-written version plus follow-up Q&A.

## How it works
- Plain HTML/CSS/JS, no build step or dependencies
- `computeAssessment()` in the client does the deterministic reasoning (severity tier, risk levels, flags)
- Claude (Claude Sonnet) is called at runtime with that assessment plus a structured prompt, and again on each follow-up question with the running conversation
- Accessible markup: semantic `form`/`fieldset`/`legend`, `aria-live` regions for generated content, managed focus on new results
- Responsive layout with light/dark theme support

## Assumptions
- Users self-report current AQI/smog level rather than the app pulling a live feed
- Recommendations are general health/lifestyle guidance, not medical advice
- Built as a single-session tool (no login, no saved history) for this submission round

## Run it
- Open `index.html` directly in a browser, or
- Use the deployed link included with this submission

## How this maps to the evaluation criteria
- **Problem Statement Alignment** — covers families *and* communities (community-actions section), and location is an actual input to the logic, not just a label (see regional smog-driver lookup above).
- **Code Quality** — decision logic (`computeAssessment`) is a pure function of its inputs, separated from DOM reading (`getFormInputs`), so it's readable and independently testable.
- **Security** — no external scripts or exposed keys; every piece of AI-generated or user-entered text is HTML-escaped (`esc`/`inline`) before being inserted into the page, preventing injected markup from the model or from form input.
- **Efficiency** — single small file, no heavy loops, targeted DOM updates (not full re-renders), no network calls beyond the one AI call per turn.
- **Testing** — `tests.html` runs 10 assertions against the rule engine and renderer (severity tiers, flags, regional notes, markdown rendering, HTML escaping). Open it directly in a browser to see pass/fail results.
- **Accessibility** — semantic `form`/`fieldset`/`legend`, `aria-live` regions on generated content, managed focus on new results, high-contrast color tokens in both light and dark mode.

## Gen AI services used
Claude (Anthropic), called live from the browser for the initial personalized plan and for each follow-up question, using a rule-computed assessment as grounding context rather than a single unstructured prompt. Claude was also used throughout development to design and build the codebase.
