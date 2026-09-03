# Japan 2026

Trip-companion app for the Japan 2026 trip: a single self-contained `index.html`
"field guide" — per-day itineraries, tappable Google Maps walking tours, packing
and shopping checklists, and a shareable trip log. No build step, no dependencies;
hosts free on GitHub Pages and works offline once loaded.

## Building the app

This repo has the **travel-field-guide-app** Claude Code skill installed at
`.claude/skills/travel-field-guide-app/`. Open a Claude Code session here and
describe the trip (cities, dates, hotels, activities) — the skill picks up
automatically and builds `index.html` from `assets/example-app.html`, following
the data model in `references/architecture.md`, the design system in
`references/design-system.md`, and the safe-edit workflow in
`references/editing-workflow.md`.

## Status

Skill installed; trip data not yet entered. Next step: provide the itinerary
(cities/dates/hotels) so the app can be built.

## Deploying

Once `index.html` exists at the repo root: push it, enable GitHub Pages
(Settings → Pages → Deploy from branch), and share the resulting URL. Add to
Home Screen on a phone for an offline, app-like experience.
