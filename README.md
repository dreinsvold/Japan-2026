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

Built. `index.html` covers the full Oct 14–27, 2026 itinerary (Tokyo → Yokkaichi
→ Yunoyama → Kyoto → Hiroshima & Miyajima → Tokyo), all 6 hotels, the rail plan,
luggage-forwarding hard anchors, and Japan-appropriate packing/shopping lists.

A few things worth double-checking closer to departure (the app calls these out
too): Miyajima Ropeway and Gozaisho Ropeway operating status, and the outbound
flight number (not in the source trip pack). The Lists → Shopping categories
are reasonable defaults (drugstore, snacks, Kyoto crafts, Hiroshima/Miyajima)
since the trip pack didn't include a per-traveler list — ask Claude to
customize them if you'd like.

## Deploying

Once `index.html` exists at the repo root: push it, enable GitHub Pages
(Settings → Pages → Deploy from branch), and share the resulting URL. Add to
Home Screen on a phone for an offline, app-like experience.
