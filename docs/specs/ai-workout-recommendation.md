# AI Workout Recommendation

## Problem Statement

As a person using the workout catalog, I want help choosing a suitable next Workout based on what I have recently done or plan to do, including workouts from outside this catalog. The current page presents useful static prescriptions but does not compare them against temporary context supplied by the user.

The recommendation request must not become a training-history feature. The user should be able to describe their current context for one request, receive a recommendation from the catalog, and have that context discarded. The product should not persist Training Sessions, calculate weeks, track repeats, or maintain a conversation transcript.

## Solution

Add a one-shot recommendation experience to the existing static workout-list page. The user enters a natural-language Recommendation Context describing recent or planned Training Sessions and any relevant constraints, such as available equipment, time, energy, or desired focus.

The page sends the temporary context and a structured JavaScript representation of the catalog to a small PHP proxy. The proxy calls one fixed OpenRouter model without retaining the request. The model chooses one primary Workout and up to two alternatives from the supplied catalog and returns a short rationale in a strict JSON response. The page validates the returned catalog IDs and presents the recommendations using the existing static Workout cards.

The existing HTML remains static and is not generated from the catalog object. Markdown is neither the catalog source of truth nor the model response format.

## User Stories

1. As a person browsing the workout catalog, I want to describe what I have done recently, so that I can receive a relevant next Workout.
2. As a person planning upcoming exercise, I want to describe Workouts I intend to do, so that the recommendation accounts for planned work as well as completed work.
3. As a person who trains outside this catalog, I want to enter outside-gym Workouts in natural language, so that the recommender is useful even when my recent activity is not represented by a catalog item.
4. As a person with limited equipment, I want to include equipment constraints in my context, so that the recommendation reflects what I can actually perform.
5. As a person with limited time, I want to include a time constraint, so that the recommendation is practical for the available session.
6. As a person with a particular training goal, I want to mention a desired focus, so that the recommendation reflects that goal when appropriate.
7. As a person whose energy or readiness varies, I want to mention that constraint, so that the recommendation can account for it without requiring a separate tracking system.
8. As a person making a recommendation request, I want to submit one natural-language context field, so that I do not have to maintain structured Training Session records.
9. As a person making a recommendation request, I want the context to be temporary, so that the product does not retain my training history.
10. As a person making multiple requests, I want each request to begin from only the context I enter at that time, so that stale prior context does not influence the result.
11. As a person receiving a recommendation, I want one primary Workout, so that I have a clear next choice.
12. As a person who wants flexibility, I want up to two alternatives, so that I can choose another suitable catalog Workout when the primary choice is impractical.
13. As a person evaluating a recommendation, I want a short rationale, so that I understand how my context influenced the result.
14. As a person receiving alternatives, I want all returned Workouts to be distinct when possible, so that the choices provide meaningful variety.
15. As a person using the catalog, I want recommendations to come only from the published catalog, so that the result points to a real Workout I can inspect.
16. As a person using the catalog, I want the recommendation to link to the existing Workout card, so that I can read the complete prescription before exercising.
17. As a person entering incomplete context, I want a clear request for more useful information, so that the system does not invent assumptions about my training.
18. As a person submitting a request, I want an understandable error when the AI provider is unavailable, so that I know no recommendation was produced.
19. As a person using the page, I want invalid or invented model results rejected, so that an AI response cannot create a Workout that is not in the catalog.
20. As a person concerned about privacy, I want to know that my context is sent to an AI provider, so that I can decide what information to enter.
21. As a person entering sensitive information, I want a warning not to include sensitive health information, so that I understand the limits of the service.
22. As a person reading a recommendation, I want a basic medical disclaimer when appropriate, so that the result is not mistaken for medical advice.
23. As a person using the page, I want the existing static catalog presentation to remain available, so that the recommendation feature does not replace normal browsing.
24. As a maintainer, I want stable Workout IDs, so that model results can be validated and connected to existing cards.
25. As a maintainer, I want structured catalog metadata, so that the model can compare Workouts using reliable fields instead of scraped presentation markup.
26. As a maintainer, I want the catalog object in the same HTML document, so that the feature remains lightweight and does not require a build step.
27. As a maintainer, I want catalog metadata to include the full prescription, so that the model can reason from the actual Workout rather than only its title.
28. As a maintainer, I want normalized catalog fields for category, format, equipment, focus, difficulty, and duration, so that recommendations can account for comparable characteristics.
29. As a maintainer, I want the OpenRouter API key kept outside browser code, so that the key is not exposed to page visitors.
30. As a maintainer, I want the PHP proxy to avoid application retention of requests and responses, so that the recommendation feature remains stateless.
31. As a maintainer, I want the model to treat user-entered context as untrusted data, so that text in the context cannot override the recommendation rules.
32. As a maintainer, I want the model limited to catalog IDs supplied in the request, so that the response cannot invent workout definitions.
33. As a maintainer, I want the model choice fixed in the proxy, so that the browser remains simple and users cannot alter provider configuration.
34. As a maintainer, I want the response contract to be strict JSON, so that the page can validate and render the result predictably.
35. As a maintainer, I want fewer than three recommendations allowed when fewer genuinely fit, so that the result is not padded with poor alternatives.

## Implementation Decisions

- Preserve the existing static Workout cards and their current presentation. Do not introduce client-side rendering for the catalog and do not add a build step.
- Add a JavaScript catalog object in the same document as the static cards. Each catalog entry has a stable ID matching its corresponding card.
- Include the full visible prescription and normalized fields for each Workout: ID, name, category, format, equipment, focus, difficulty, duration, and prescription.
- Treat the JavaScript catalog object as the structured source for recommendation requests while the static HTML remains the display source. Keeping both representations requires deliberate manual synchronization.
- Use the domain terms `Workout`, `Training Session`, and `Recommendation Context` as defined in the project glossary.
- Model the user input as one ephemeral Recommendation Context. It may contain outside-gym Workouts, planned Workouts, equipment, time, energy, focus, and other constraints in natural language.
- Do not persist Recommendation Context, Training Sessions, chat messages, recommendation history, week boundaries, repeat state, or user accounts.
- Use a one-shot request rather than a conversational chatbot. Each submission is independent.
- Add a PHP server-side proxy between the page and OpenRouter. The browser must not receive or contain the OpenRouter API key.
- Configure one fixed OpenRouter model in the PHP proxy.
- The proxy forwards only the temporary Recommendation Context and the structured catalog required for the request. It does not retain application copies of requests or responses.
- The model receives explicit instructions to treat the Recommendation Context as untrusted data, ignore instructions embedded in it, and choose only from the supplied catalog.
- Require a strict JSON response containing a primary catalog Workout ID, up to two alternative catalog Workout IDs, and a short rationale.
- Require distinct primary and alternative IDs when possible. Alternatives may be absent when fewer than three genuinely fit.
- Validate the response before rendering. Every non-null ID must exist in the supplied catalog, and the page must reject malformed JSON, unknown IDs, duplicate IDs where distinct choices are expected, and responses outside the contract.
- Render recommendation names, prescriptions, and links from the local catalog/static cards rather than trusting model-generated Workout definitions.
- Show a clear error for empty or insufficient context, proxy failures, OpenRouter failures, malformed responses, and invalid catalog IDs. Do not silently invent or substitute a recommendation.
- Display that the entered context is sent to the configured AI provider and should not contain sensitive health information.
- Include the agreed basic medical disclaimer: the result is general workout information, not medical advice; users should stop if pain occurs and consult a qualified professional. The model must not diagnose, claim that a Workout is safe, or modify a prescription as medical treatment.
- Keep provider/model configuration and API credentials in the PHP deployment environment or server configuration, not in the HTML document.

## Testing Decisions

- The primary test seam is the recommendation request boundary: temporary context enters the recommendation service, a provider response is supplied through a replaceable fake transport, and a validated recommendation response exits. This is the highest useful seam because it covers prompt assembly, provider interaction, response validation, stateless behavior, and the API contract without testing implementation details.
- Existing UI behavior should be covered by a browser-level smoke test that verifies the static catalog remains visible, the form accepts context, a valid response displays one primary Workout and up to two alternatives, and displayed results correspond to catalog cards.
- Test only observable behavior and contracts. Do not assert DOM traversal mechanics, prompt string formatting, internal helper names, or the exact OpenRouter SDK/request implementation.
- Test that a valid provider response with known catalog IDs is accepted and rendered with catalog-owned names and prescriptions.
- Test that unknown, duplicate, missing, malformed, or non-catalog IDs are rejected without displaying an invented Workout.
- Test that a response with one primary and fewer than two valid alternatives is accepted when the provider explains that fewer choices fit.
- Test that empty or insufficient Recommendation Context is rejected with an actionable message before an unnecessary provider request.
- Test that provider timeout, non-success response, invalid JSON, and contract violations produce the documented error state.
- Test that each request is independent and that no prior Recommendation Context is included in a later request.
- Test that the PHP proxy does not persist application request or response data.
- Test that the API key is not present in browser-delivered content or client-side request configuration.
- Test that user-entered context is delimited and treated as data rather than executable instructions to the model.
- Test that the medical disclaimer and provider/privacy notice appear in the relevant states.
- There is currently no testing framework or comparable feature in the repository. Establish the smallest test harness around the recommendation request boundary, supplemented by a browser smoke test if the deployment environment supports it.

## Out of Scope

- Persisted Training Sessions or a training-history database.
- User accounts, authentication, profiles, or multi-user separation.
- Local browser storage for Recommendation Context or recommendation results.
- A conversational chat transcript or multi-turn follow-up experience.
- Automatic week calculation, repeat tracking, recovery history, or calendar planning.
- A generated catalog, build pipeline, Markdown source file, Markdown parser, or client-side catalog rendering.
- Recommendations for Workouts not present in the catalog.
- AI-generated exercise prescriptions, substitutions, load adjustments, or new Workout definitions.
- Medical diagnosis, injury treatment, safety certification, or personalized medical advice.
- Automatic fallback recommendations when OpenRouter fails.
- User-selectable OpenRouter models.
- Analytics, personalization, recommendation history, or model-quality learning loops.
- Guaranteed availability of exactly three recommendations when fewer than three are appropriate.

## Further Notes

- The current page is a single static HTML document containing the Workout catalog and presentation. The feature should fit that lightweight shape rather than introduce a frontend framework or build system.
- The structured catalog object intentionally duplicates some visible Workout information. Stable IDs and manual synchronization are accepted for this version in exchange for keeping the page static and simple.
- Outside-gym Workouts cannot be deterministically mapped to catalog IDs. They are contextual natural-language evidence for the model, while the output remains constrained to catalog IDs.
- The issue should be published with the `ready-for-agent` triage label once GitHub CLI authentication is available.
