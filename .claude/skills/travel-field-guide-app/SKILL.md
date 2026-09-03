---
name: travel-field-guide-app
description: >-
  Build or edit a self-contained, offline-capable mobile travel-companion web
  app — a single index.html "field guide" for a multi-city trip, with per-day
  itineraries, tappable Google-Maps walking tours, transit connectors, packing
  and shopping checklists, per-device save, and a shareable trip log. Use this
  skill whenever the user wants a travel app, trip companion, itinerary app, or
  "field guide" to host on GitHub Pages and share with fellow travelers, OR when
  editing an existing app built from it — adding stops, days, markets, shopping
  lists, restaurants, or callouts; reordering a walking tour; splitting a driver
  day; etc. Trigger it even for casual phrasings like "turn my travel guide into
  an app," "add a stop/day/shopping list to my trip app," "reorder the walking
  tour," or "make Strasbourg a loop." It encodes the exact data model, the
  night-train design system, the walking-tour routing rules (loop or
  furthest-first), and the safe edit-and-validate workflow so nothing regresses.
---

# Travel Field-Guide App

A single-file, offline-first mobile web app that turns a multi-city trip into a
tappable "field guide." Built as one `index.html` — vanilla HTML/CSS/JS, no
build step, no dependencies, no external fonts — so it loads offline once cached
and drops straight onto GitHub Pages for a whole travel group to share.

## Start here (every time)

1. **Editing an existing app?** Work directly on that `index.html`. **Building
   fresh?** Copy `assets/example-app.html` — it is a complete, working reference
   implementation (all views, checklists, and features wired up). Adapt its data.
   It ships with a sample 12-day trip and placeholder travelers ("Traveler 1–3")
   and a placeholder shared-album link — replace all of them with the real trip.
2. **Read `references/architecture.md` before touching data.** It documents the
   `DATA` and `SHOPPING` objects, the day/stop/callout shapes, the tab/view
   system, and the `store` (localStorage) layer.
3. **Follow `references/editing-workflow.md` for every change.** It has the
   exact safe-write pattern, the `DATA` extraction regex, the emoji gotcha, and
   the Playwright validation loop. Skipping it is how the file gets corrupted.
4. **Keep the design intact** — see `references/design-system.md`.
5. After any edit: validate headlessly, then `present_files` the `index.html`
   (primary) and the README. Remind the user the **GitHub copy lags** the
   outputs file, so they must re-upload `index.html` to push changes to the group.

## The app at a glance

Five bottom tabs: **Today** (auto-detects the trip day; live "departure-board"
timetable; countdown before, complete after), **Days** (list → detail with
timetable, numbered walking tour, eat, foods, good-to-know, notes), **Places**
(hotels + city accordions), **Lists** (a switch between Packing and Shopping;
shopping has per-person switchable checklists with tap-for-price/details), and
**Info** (emergency, transport, shared photo album, and the trip-log export).

Per-device persistence via `localStorage` (wrapped so it degrades gracefully):
per-stop "Seen" toggles, per-day notes, packing checks, and shopping-buy checks.
The trip log exports what each traveler checked off as shareable text.

## Non-negotiable rules

These are the conventions that make the app feel considered. Honor them on every
edit.

### 1. Walking-tour routing — a loop, or furthest-first-then-back

Order every day's `tour.stops` as a deliberate walk **from the hotel**, one of:

- **A circle** (preferred): leave the hotel, go down one side to the furthest
  point, cross/turn, and come back the other side so you **end near the hotel**.
  E.g. Zürich: south down Bahnhofstrasse to the lake (furthest), cross the
  Limmat, back up the Niederdorf old-town side to the hotel.
- **Furthest-first**: start at the point furthest from the hotel and work back
  toward it.

**Never** leave stops "all over the place" — zig-zagging out to a far point and
back to a near one and out again. When you reorder, also fix the day's
`schedule` so its times follow the same path (no east–west backtrack). State the
loop logic in the `tour.intro`. A sensible exception: a crowd-sensitive sight
(e.g. a cathedral) can open the route "first thing" even if central — note why.

### 2. Optional stops go last and are labeled "(optional)"

Append non-essential stops (markets, extra museums, shops, a secondary
pharmacy) after the core route, with `(optional)` in the name. The core numbered
walk stays tight; optional extras don't inflate the "N stops" count logic that
already filters transit rows.

### 3. Use transit connectors between legs

When a day covers two places (a driver hop, a train between towns, a coast
extension), insert a stop object with `transit:true` between them. It renders as
a distinct connector (train icon, no number, no "Seen" toggle) and carries the
how-to-get-there detail. Don't fake it with a normal stop.

### 4. Every stop is tappable to a map

Give each real stop `url` + `addressUrl` as Google Maps **search** links:
`https://www.google.com/maps/search/?api=1&query=<URL-encoded place + city>`.
Prefer a precise address in the query; a name+city query is an acceptable
fallback. Preserve any verbatim URLs the source already provided.

### 5. Be honest in recommendations

This app is full of judgment calls (is a stop worth it, is a shop open Sunday,
does one pharmacy carry everything). Verify current facts (opening days,
transport, whether a shop exists) with search before asserting, and correct
flawed premises kindly rather than parroting them. Note weekday/opening
alignment against the actual trip dates.

### 6. Edit safely (see editing-workflow.md)

The `DATA`/`SHOPPING` blocks live inside the HTML. Editing structured data by
hand corrupts easily. Use the documented Python round-trip with an
**encode-before-write** guard and an atomic temp-file swap, then re-validate.
Never write astral emoji as `\uXXXX\uXXXX` surrogate pairs — use real characters
or `\U0001Fxxx`.

## Common tasks → where to look

- Add/split a day, add stops, reorder into a loop, add markets/shops/restaurants,
  add callouts → `references/architecture.md` (shapes) + `references/editing-workflow.md` (how).
- Add a shopping checklist / person, add price + "what it does" + shade details →
  `references/architecture.md` → "SHOPPING model".
- Change colors, the departure board, the leg accents, add a tab →
  `references/design-system.md`.
- Deploy / share / offline → tell the user: GitHub Pages (upload `index.html`,
  enable Pages), Add-to-Home-Screen for offline, one QR code for the group.

## Reference files

- `references/architecture.md` — data model (`DATA`, `SHOPPING`), tabs, views, store, features.
- `references/design-system.md` — the night-train aesthetic, tokens, components, layout.
- `references/editing-workflow.md` — safe-edit pattern, validation, gotchas, deploy notes.
- `assets/example-app.html` — a complete working app to start from or study.
