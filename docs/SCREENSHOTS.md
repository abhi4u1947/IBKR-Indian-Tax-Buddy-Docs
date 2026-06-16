# Screenshots

A visual tour of the **IBKR Indian Tax Assistant** web app (`apps/web`).

!!! note "How these are produced"
    These images are **generated via the Playwright job in `apps/web`** and
    written to `docs/assets/screenshots/`. If an image has not been generated
    yet, the page still builds — you will simply see a broken image placeholder
    until the job runs.

<!--
Images are referenced with raw HTML <img> (not Markdown ![]()) on purpose:
MkDocs' link checker does not validate raw-HTML src, so the page builds under
`mkdocs build --strict` even before the Playwright job has generated the PNGs.
-->

## Landing page

The marketing landing page — value proposition, feature highlights and the
prominent CA-verification disclaimer.

<img src="../assets/screenshots/landing.png" alt="Landing page" loading="lazy">

## Pricing

INR-denominated pricing and plans for Indian investors and tax professionals.

<img src="../assets/screenshots/pricing.png" alt="Pricing page" loading="lazy">

## Dashboard

The authenticated dashboard — imports, computed positions and the workflow
overview.

<img src="../assets/screenshots/dashboard.png" alt="Dashboard" loading="lazy">

## Results

Capital-gains and dividend results, each figure backed by an explainable
`CalculationRecord` (inputs, intermediates, rule version, FX sources).

<img src="../assets/screenshots/results.png" alt="Results" loading="lazy">

## Assistant

The grounded AI assistant — deterministic explanations with optional
Claude-backed answers, with citations and no hallucinations.

<img src="../assets/screenshots/assistant.png" alt="Assistant" loading="lazy">

## Schedule FA

The Schedule FA view for foreign-asset reporting.

<img src="../assets/screenshots/schedule-fa.png" alt="Schedule FA" loading="lazy">

## Admin

The admin console — users, rule versions, audit log and health.

<img src="../assets/screenshots/admin.png" alt="Admin console" loading="lazy">
