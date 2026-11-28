# ClearAir — Personal Air-Quality Guidance

**Live application:** https://anik-2007.github.io/clearair-promptwars/  
**Browser test runner:** https://anik-2007.github.io/clearair-promptwars/tests.html  
**GitHub repository:** https://github.com/anik-2007/clearair-promptwars

ClearAir is a static, browser-based air-quality guidance application. It combines a city-level air-quality lookup with household and commute inputs to generate practical exposure-reduction suggestions. Its core recommendation plan is rule-based and works independently of the optional AI explanation.

## Features

- City search through the Open-Meteo Geocoding API.
- Current US AQI and available PM2.5/PM10 readings through the Open-Meteo Air Quality API.
- Manual AQI-category fallback when live retrieval is unavailable.
- Household-aware recommendations using children, older adults, respiratory conditions, outdoor work, home type, purifier availability and commute mode.
- Regional seasonal context for Delhi-NCR, Lahore and Dhaka. These are general possible contributors, not live source-apportionment results.
- Optional, user-initiated Claude explanation when the host environment provides the integration. AI is not required for the core plan.
- Responsive layout, keyboard focus indicators, semantic form labels, live status regions and reduced-motion styling.

## Repository structure

| File | Responsibility |
|---|---|
| `index.html` | Complete application, API requests, assessment rules and plan rendering. |
| `tests.html` | Dependency-free browser test runner that loads `index.html` in an iframe and tests its read-only `window.ClearAirTestAPI`. |
| `README.md` | Project documentation, architecture, setup, tests and limitations. |

Keep all three files in the repository root. No build step or package installation is required.

## Run and test

1. Clone or download the repository.
2. Serve the folder using any static HTTP server (recommended), or deploy it with GitHub Pages.
3. Open `index.html` to use ClearAir.
4. Open `tests.html` to run the browser test suite.
5. Confirm the summary reports all tests passing before submitting.

The test runner executes **67 deterministic checks** against functions exported by the actual application file. It does not make live AQI requests or call the optional AI service. Passing these tests verifies the listed deterministic rules; it does not prove that every external service, browser, screen reader or AI integration will work in every environment.

## Architecture and data flow

1. **Resolve location:** the entered city is sent to Open-Meteo Geocoding to obtain a location and coordinates.
2. **Retrieve air quality:** coordinates are sent to Open-Meteo Air Quality for current US AQI and available particulate values.
3. **Validate and classify:** the AQI value is checked and mapped to one of six displayed categories.
4. **Assess exposure:** pure JavaScript functions derive household, indoor, outdoor-work and commute signals from the selected inputs.
5. **Generate the core plan:** deterministic rules build routine, indoor, commute, community and household-impact recommendations.
6. **Optional explanation:** only after the user requests it does the app attempt the host-provided Claude integration. The AI is instructed to explain the existing assessment, not invent or replace its measurements.

## Code quality and efficiency

The assessment and plan-building functions are separated from most UI event handling and are exposed through a frozen, read-only test API. The AQI lookup uses a five-minute in-memory cache, limits the cache to 30 entries, applies a 12-second request timeout and shares an in-flight request for the same normalized city. A stale lookup result is ignored when the user changes the city or chooses the manual AQI path. These measures apply within the current browser page session; the cache is not persistent storage.

## Security

- Dynamic recommendation and AI text is inserted using safe DOM text operations; the Markdown-like AI renderer creates only a small set of elements and treats supplied text as text rather than HTML.
- City input has a length limit and is passed as URL-encoded query data.
- Network requests use a timeout and check HTTP response status before parsing JSON.
- The test surface exposes selected functions only and is frozen; it is not an application state mutation API.
- Open-Meteo data retrieval does not require a user-provided API key. The optional AI integration depends on the host environment.

## Accessibility

The page uses a skip link, semantic main/form/section structure, associated labels, fieldset/legend grouping, visible keyboard focus styles, polite live regions for dynamic content and a reduced-motion media query. Automated checks do not replace manual keyboard, zoom, contrast and screen-reader testing.

## Evaluation alignment

| Evaluation area | Evidence in this project |
|---|---|
| Code Quality | Separated assessment helpers, configuration constants, named functions and documented file structure. |
| Security | Text-safe DOM rendering, input length constraint, encoded URL parameters and bounded network timeout. |
| Efficiency | In-memory TTL/LRU-style cache, maximum entry count, duplicate in-flight request sharing and request timeout. |
| Testing | `tests.html` exercises the real app's exposed pure functions with AQI boundaries, risk derivation, plan content, regional notes and safe-rendering checks. |
| Accessibility | Semantic labels and landmarks, keyboard focus, skip link, live regions and reduced-motion support. |
| Problem Alignment | Live city-level AQI, household and commute-specific recommendations, manual fallback and regional seasonal context. |

## Data sources, assumptions and limitations

- **Open-Meteo Geocoding API:** city-name lookup and coordinates.
- **Open-Meteo Air Quality API:** current US AQI and available PM2.5/PM10 values.

Air-quality values are time- and location-dependent. A city-level result may not represent conditions at a particular street, building or indoor space. The app uses the US AQI scale for its six categories. Regional pollution notes are contextual background and do not attribute a particular live reading to a specific source. Household risk labels are planning signals, not clinical risk predictions. ClearAir is educational and is not a medical diagnosis or treatment tool. Follow local public-health alerts and consult a qualified clinician for personal medical concerns.

The optional AI explanation may be unavailable outside a compatible host environment. The core deterministic plan remains available without it.

## Deployment

This is a static website suitable for GitHub Pages. Publish the repository root and ensure `index.html`, `tests.html` and `README.md` retain these exact filenames. After each code change, verify both the live app and the test runner URLs above.
