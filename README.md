# ClearAir — Urban Air Quality Assistant

**PromptWars (Hack2skill) — Challenge 6: Urban Air Quality Assistant**

## Problem
Families in cities with severe winter smog need practical, personalized guidance based on actual current conditions — not generic tips, and not a number the user has to guess themselves.

## Design
The live AQI reading is the visual centerpiece: a color-coded badge (standard EPA green→maroon scale) with a severity gauge, the resolved location, and the exact PM2.5 reading and source — so real data is immediately legible, not buried in text. Each plan section has an icon for quick scanning. "Young children" and "senior citizen" are now separate checkboxes rather than one bundled option, matching the distinct personas named in the challenge brief (families with young children, senior citizens, working professionals).

## Live air quality data
On submit, the app geocodes the entered city and fetches **real-time air quality** (US AQI, PM2.5, PM10) via the free, public **Open-Meteo Air Quality API** — no API key, fetched directly from the browser. The fetched reading is shown on screen (location resolved, US AQI, PM2.5, source, "just now") so it's visibly live data, not a static or generated value.

**Why Open-Meteo instead of OpenAQ/CPCB:** OpenAQ's own documentation states that direct client-side/browser calls to its API are not possible due to CORS policy, and recommends a backend proxy — which this project, a static single-page app with no server, doesn't have. CPCB has no public, CORS-enabled API suitable for direct browser use either. Open-Meteo is free, keyless, and explicitly built for direct client-side consumption, so it was chosen as the real-time data source that could actually work without a backend.

If the city can't be found or the live fetch fails for any reason, the app reveals a manual AQI selector as a fallback so the demo never dead-ends — but live fetch is the default, primary path for every normal use.

**Sample cities to test:** Delhi, Beijing, London, Jakarta, Lahore, Dhaka, Los Angeles — try a clean city (e.g. "Reykjavik") versus a polluted one (e.g. "Delhi" in winter) to see the plan change with real conditions.

## Approach
Once the live AQI tier is known, a rule engine computes risk: commute exposure, indoor air risk, location-specific smog drivers (Delhi/Lahore/Dhaka regional patterns), and household priority flags (children/elderly, respiratory conditions, outdoor work, vehicle access). The plan has five sections: today's low-emission routine, indoor air quality hacks, sustainable commute alternatives, community actions (carpools, pollution reporting — addressing *communities*, not just the household), and why it matters for this specific household.

## Gen AI integration
A separate, clearly-labeled "✨ Get a richer AI explanation" button is fully opt-in. Clicking it sends Claude the rule engine's live-data-derived assessment (AQI tier, risk levels, flags) as ground truth, and Claude explains and acts on it — the output changes with every different input, nothing is hardcoded. A follow-up Q&A box lets the user ask specific questions with full conversation context. This is opt-in rather than automatic so the core app works instantly for everyone, with Claude enriching rather than gating the experience.

## Deployment note
This app makes outbound network calls (Open-Meteo) and must run on a host that allows that — **GitHub Pages or Netlify, not the claude.ai artifact preview**, which blocks all outbound fetches by platform policy. The live AQI fetching simply cannot function there; only a plain static host works for this build.

## How this maps to the evaluation criteria
- **Problem Statement Alignment** — real-time external API data drives every recommendation (not static/generated values), covers families *and* communities, and location genuinely changes the output.
- **Code Quality** — decision logic (`deriveFacts`/`computeAssessment`) is a pure function of its inputs, separated from the live-fetch and DOM-reading code.
- **Security** — no API keys needed (Open-Meteo is keyless), no external scripts, all dynamic/AI/user text is HTML-escaped before insertion.
- **Efficiency** — two lightweight API calls per submission, no polling, no unnecessary network calls.
- **Testing** — `tests.html` runs assertions against the rule engine, the US-AQI-to-tier mapping, and the renderer.
- **Accessibility** — semantic form/fieldset/legend, aria-live regions, managed focus, high-contrast theming.

## Code quality & efficiency notes
Every major function has a one-line doc comment explaining its purpose, and the script opens with a short architecture comment describing the full data flow. `fetchLiveAQI()` caches results per city for 5 minutes, so re-checking the same city (e.g. during a demo) skips redundant network calls while still reflecting genuinely current conditions.

## How it works
- Plain HTML/CSS/JS, no build step or dependencies
- `fetchLiveAQI()` geocodes the city then queries Open-Meteo's Air Quality API
- `deriveFacts()`/`computeAssessment()` turn the live AQI tier plus form inputs into a structured risk assessment
- Claude (Claude Sonnet) is called only when the user opts in, using that assessment as grounding

## Assumptions
- Recommendations are general health/lifestyle guidance, not medical advice
- Built as a single-session tool (no login, no saved history) for this submission round
- Open-Meteo's CAMS-derived air quality model is used as the live data source in place of ground-station networks (OpenAQ/CPCB) that don't support direct browser access

## Run it
Open `index.html` in a browser, or use the deployed GitHub Pages / Netlify link included with this submission.
