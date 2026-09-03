# Design System — "Continental Pullman" night-train field guide

The look is a warm, printed night-train field guide: ticket-stock paper, deep
midnight-teal chrome, brass accents, and a per-country accent that shifts as the
trip moves between legs. Keep this identity when editing; don't reach for a
generic UI kit. Everything is defined in the `<style>` block of the app.

## Core tokens (CSS variables)

- `--paper: #FAF6EE` (ticket stock), `--paper-2` (slightly deeper card paper)
- `--ink` (near-black warm text), `--muted` (secondary text), `--line` (hairline)
- `--teal / #0E2A33` chrome (top bar, nav) — also the `theme-color` meta
- `--brass: #B8893E` / highlight `#C99A4A` — pricetags, small accents
- `--accent` — **set at runtime per view** via `setAccent(color)`

## Per-leg accent colors (`LEGS`)

Each trip leg gets a country accent; the current day sets `--accent` to its leg:

- Belgium gold `#C8941E` · Luxembourg slate-blue `#3E6B8C` · Alsace red `#A8423C`
  · Switzerland green `#2F6B4F` · transit/neutral `#7A6A52`

`setLegPill({name, flag})` renders the little country pill in the top bar. When
building a new trip, define a `LEGS` map (day-number → `{color, name, flag}`) and
have Today/day-detail call `setAccent(leg.color)`.

## Typography

System font stacks only — **no web fonts** (offline reliability):
- serif for display/headlines, sans for body, monospace (`--mono`) for times,
  eyebrows, pricetags, and the departure board.

## Signature components

- **Departure board**: the day `schedule` renders as a vertical timetable with
  monospaced times on a track — the app's signature element. Keep it for
  schedules.
- **`.section-head`**: eyebrow (mono, uppercase) + `.h-section` title + `.sub`.
- **`.card.pad`**: the standard rounded paper card.
- **`.acc` / `<details>`**: accordions (Places cities, Packing lists).
- **`.check` / `.box` / `.lbl`**: a checkbox row (packing + shopping). `.check.on`
  fills the box with `--accent`.
- **`.seg`**: segmented switcher (Packing/Shopping, and the person switcher). It
  wraps (`flex-wrap`) so 3–4 options fit on a phone.
- **`.stop` / `.num` / `.maplink`**: walking-tour stop with numbered badge and
  "📍 Open in Maps". `.seentog` is the per-stop Seen pill.
- **`.leg` / `.leg-ic`**: transit connector row (dashed train icon, tinted).
- **`.tier-head`**: mono accent sub-header inside shopping lists.
- **`.buydetails`**: the expandable price/does/shade panel; shown via
  `.check.open + .buydetails`.
- **`.pricetag`**: mono brass, used for counts and shopping prices.

## Layout rules

- Mobile-first, ~390px design width; a phone shows ~6–8 lines at once.
- Minimal formatting; let the paper + accents carry the style.
- Bottom nav: keep to 5 tabs so labels don't wrap. Add capability via a
  segmented sub-switch, not more tabs.
- Respect `prefers-reduced-motion` (already handled).
- Every screen sets an accent so the top bar/leg pill stays coherent; checklist
  views use the transit neutral for Packing and the list's own accent for
  Shopping.
