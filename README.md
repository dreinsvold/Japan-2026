# Japan 2026

Trip-companion app for the Japan 2026 trip (Oct 14–27): a single self-contained
`index.html` "field guide" — per-day itineraries, tappable Google Maps walking
tours, a packing checklist, and a shareable trip log. No build step, no
dependencies; hosts free on GitHub Pages and works offline once loaded.

## Building the app

This repo has the **travel-field-guide-app** Claude Code skill installed at
`.claude/skills/travel-field-guide-app/`. Open a Claude Code session here and
describe the trip (cities, dates, hotels, activities) — the skill picks up
automatically and builds `index.html` from `assets/example-app.html`, following
the data model in `references/architecture.md`, the design system in
`references/design-system.md`, and the safe-edit workflow in
`references/editing-workflow.md`.

## Status

`index.html` is built from the authoritative master itinerary (8 overnight
stays, 4 booked Shinkansen reservations, 2 luggage-forwarding shipments, 2
booked dinners, the Hiroshima museum ticket). A few things are intentionally
left as the source itinerary flagged them:

- No hotel phone numbers or booking confirmation numbers (not provided).
- No private address for the Yokkaichi friends/family stay.
- The Sushi Fujioka and Gion Fukushi addresses were found via web search, not
  from the actual reservation confirmations — verify against your own booking
  before relying on them.
- `weather` is left blank per day — a forecast checked now would be stale by
  travel time; re-check closer to departure instead.
- Remaining operational to-dos (QR tickets, luggage shipment execution, the
  Shikanoyu taxi booking, etc.) are listed in the source itinerary's
  "Remaining actions" section and aren't tracked as checkboxes in the app.

To update the trip (add a stop, fix a time, reorder a walking tour), open a
Claude Code session here and describe the change — the **travel-field-guide-app**
skill at `.claude/skills/travel-field-guide-app/` picks it up automatically.

## Deploying

Once `index.html` exists at the repo root: push it, enable GitHub Pages
(Settings → Pages → Deploy from branch), and share the resulting URL. Add to
Home Screen on a phone for an offline, app-like experience.
