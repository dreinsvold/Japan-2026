# Architecture

The whole app is one `index.html`: `<style>` (design tokens + components), then
a `<script>` holding two data objects (`DATA`, `SHOPPING`), a `TABS` list, a
`store` layer, view functions, and a `render()` loop. Study
`assets/example-app.html` alongside this.

## Table of contents
- The `DATA` object
- Day object
- Stop object + transit rows
- Callouts, eat, foods, good
- The `SHOPPING` object
- Tabs, state, and views
- The `store` (persistence) layer
- Features (Seen, notes, export, shared album)

## The `DATA` object

`const DATA = { meta, glance, hotels, drivers, days, cities, restaurants, foods, transport, packing, emergency };`

**Field names below are exactly what `assets/example-app.html` uses. Copy them
verbatim — a typo here (e.g. `startISO` for `tripStart`) silently breaks the Today
tab, which is this skill's known failure mode.**

- `meta`: `{ title, subtitle, dates, tripStart, tripEnd }` — `tripStart`/`tripEnd`
  are ISO dates (`"2026-07-10"`) and drive the Today hero and all trip-day math.
- `glance`: `{ route:[{nights, city, hotel, country}], flights:[{k, v}],
  key:[string] }` — `key` is the list of confirmed items.
- `hotels`: `[{ name, address, mapUrl, phone, notes }]` — note **`mapUrl`**, not
  `url`; walking-tour stops use `url`/`addressUrl` instead.
- `drivers`: `[{ title, when, detail:{label: value, …}, stops:[stop objects] }]` —
  `detail` is a free-form ordered map (Pickup, Drop-off, Payment, Departure…).
- `days`: the core array (see below).
- `cities`: `[{ city, links:[{label, t, u}], attractions:[{t, u, note}],
  restaurants:[{t, u, note}] }]` — the Places accordions. `t` = title, `u` = URL
  (may be `null` for a non-linked row).
- `restaurants`: `[{ city, rows:[{cat, rec, price}] }]`.
- `foods`: `[{ country, items:[{name, desc}] }]` (top-level; distinct from a day's
  `foods`, which is a flat `[{name, desc}]`).
- `transport`: `{ tables:[{name, tables:[[row, …]], bullets:[string]}],
  notes:[{title, body}], bullets:[string] }` — inner `tables` are arrays of rows,
  first row is the header.
- `packing`: `[{ title, items:[string] }]` — the Packing checklists.
- `emergency`: `{ numbers:[{country, general, police, amb}],
  hospitals:[{city, hospital, embassy}], money:[{k, v}] }`.

## Day object

```
{
  n: 3, weekday:"Sunday", date:"July 12", iso:"2026-07-12", short:"Sun, Jul 12",
  headline:"Day Trip — Bruges", subtitle:"Canals, chocolate & a brewery",
  overview:"…", weather:"…", walking:"~5 km", transport:"…",
  schedule:[ { time:"8:00", plan:"Walk to the station" }, … ],   // departure board
  tour:{ intro:"…loop logic…", stops:[ …stop objects… ] } | null, // null on pure-transfer days
  callouts:[ { title:"🚋 …", body:"…" } ],
  eat:[ { cat:"🍽 Lunch", rec:"Place", notes:"…" } ],
  foods:[ { name:"Waterzooi", desc:"…" } ],
  good:{ photo:"…", coffee:"…", restrooms:"…", rainy:"…", packing:"…" }
}
```

- `n`/`weekday`/`date`/`iso`/`short` are **position-specific**. When you swap two
  days' content, keep these fields with the slot and move only the content fields
  (`headline…good`). Verify weekday-dependent notes (markets, shop closures)
  still hold after a swap.
- Driver/transfer days often have `tour:null` and rely on `callouts`. You can add
  a `tour` with `stops` to a driver day to get map pins + Seen toggles; the
  renderer guards with `if(d.tour && d.tour.stops.length)`.

## Stop object + transit rows

```
{ name:"Rozenhoedkaai", url:"<maps search url>", address:"…, 8000 Brugge",
  addressUrl:"<maps search url>", desc:"1–4 sentences of shareable detail." }
```

Transit connector (between two legs):
```
{ transit:true, name:"Coast → Ghent by train", url:"<maps url>",
  desc:"Kusttram to Ostend (~20 min), IC to Gent-Sint-Pieters (~45 min)…" }
```
Transit rows render as a tinted "leg" connector — no number, no Seen toggle — and
are excluded from the "Self-guided · N stops" count
(`d.tour.stops.filter(s=>!s.transit).length`).

Optional stops: put `(optional)` in the name and place them after the core route.

## Callouts, eat, foods, good

- `callouts`: `{title, body}`. Titles usually start with one emoji. Use for tips,
  ticket advice, "evening is free," a mini-plan for a driver-day stop, etc.
- `eat`: `{cat, rec, notes}` (cat starts with an emoji).
- `foods`: `{name, desc}`.
- `good`: fixed keys `photo, coffee, restrooms, rainy, packing`.

## The `SHOPPING` object

Separate from `DATA` (a top-level `const SHOPPING`), because it holds emoji-heavy
tier titles. Powers the Lists → Shopping switcher.

```
const SHOPPING = { lists:[
  { id:"traveler1", label:"Traveler 1", flag:"🇫🇷", accent:"#A8423C",
    note:"Strasbourg pharmacy — …",
    tiers:[
      { title:"⭐ Must", items:[
        { n:"Caudalie Vinoperfect Serum", p:"€45–50",
          d:"Brightening serum that fades dark spots.", s:"optional shade note" },
        …
      ]},
      …
    ]},
  { id:"traveler2", label:"Traveler 2", … },
  { id:"traveler3", label:"Traveler 3", … },
  { id:"swiss", label:"Switzerland", flag:"🇨🇭", accent:"#2F6B4F", … }
]};
```

- Each list: `id` (stable — used in storage keys `buy:<id>:<tier>:<item>`; don't
  rename ids or you orphan saved checks — change `label` instead), `label`
  (displayed), `flag`, `accent` (list-colored), `note`, and `tiers`.
- Each item is an object: `n` name, `p` price range, `d` what-it-does, optional
  `s` shade. Tapping a row expands `p`/`d`/`s`; the checkbox toggles bought.
- Keep pharmacy-list accents France-red (`#A8423C`) and a Switzerland-style list
  green (`#2F6B4F`); the view calls `setAccent(list.accent)`.
- Reality check to bake in: big parapharmacies carry all the major French derm
  brands — one shop covers a whole list; don't route people to several "for
  different brands." Flag only genuinely stock-dependent items.

## Tabs, state, and views

- `const TABS = [{id,label,svg}]` — 5 tabs: today, days, places, lists, info.
- `const LEGS = { key:{name, flag, color} }` plus `const DAY_LEG = { dayNumber: legKey }`
  — this pair drives the per-country accent and the top-bar leg pill. Every day
  number must have an entry in `DAY_LEG`.
- `const state = { tab, day, shopList, listMode }`. **`shopList` must default to a
  real `SHOPPING.lists[].id`** — if you rename the first list's id, update this too
  (the views fall back to `lists[0]`, but keep them in sync anyway).
- `render()` switches on `state.tab` to a `viewX()` that returns an HTML string.
- Lists tab: `viewLists()` shows a Packing/Shopping segment (`state.listMode`),
  then calls `packBody()` or `shopBody()`. Shopping has its own person switcher
  (`state.shopList`) via `switchShop(id)`.
- Adding a tab: append to `TABS`, add a `case` in `render()`, write the `viewX()`.
  Keep to ~5 tabs so labels don't wrap; prefer a sub-switch (segmented control,
  class `.seg`) over a 6th tab.

## The `store` (persistence) layer

`store.get(key, fallback)` / `store.set(key, val)` wrap `localStorage` in
try/catch with an in-memory fallback, and expose `store.ok`. Keys in use:
- `pk:<listIdx>:<itemIdx>` — packing
- `buy:<listId>:<tierIdx>:<itemIdx>` — shopping
- `note:day:<n>` — per-day notes
- `seen:<dayN>:<stopIdx>` — per-stop Seen toggles

All state is per-device (each traveler keeps their own). True cross-device sync
would need a backend (e.g. Firebase) and is intentionally not included.

## Features

- **Seen toggles** on every walking-tour stop (`toggleSeen`), tinting done stops.
- **Notes** textarea per day (persist on input).
- **Trip-log export** on Info (`buildTripLog` → copy/download/preview) — lists
  each traveler's checked-off places + notes as shareable text.
- **Shared photo album** card on Info (a link out to a cloud folder).
- **Departure-board** schedule rendering on Today and in day detail.
