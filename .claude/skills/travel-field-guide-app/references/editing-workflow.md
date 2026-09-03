# Editing Workflow

The `DATA` and `SHOPPING` blocks live inside the HTML. Hand-editing structured
data is error-prone, so use these patterns and validate every change.

## Editing `DATA` (the safe round-trip)

Extract → parse → modify → re-serialize → **encode-guard** → atomic write.

```python
import re, json, os
path='…/index.html'
html=open(path,encoding='utf-8').read()
m=re.search(r'(const DATA\s*=\s*)(\{.*?\})(;)', html, re.DOTALL)   # non-greedy; DATA is pure JSON so first "};" is its true end
prefix,body,suffix=m.group(1),m.group(2),m.group(3)
data=json.loads(body)

# … mutate data …  (append stops, reorder, add callouts, swap day content, etc.)

newbody=json.dumps(data, ensure_ascii=False)
assert '</script>' not in newbody
html2=html[:m.start()]+prefix+newbody+suffix+html[m.end():]
payload=html2.encode('utf-8')            # ENCODE FIRST — raises here on bad chars, before the file is touched
tmp=path+'.tmp'
open(tmp,'wb').write(payload)             # write bytes to temp
os.replace(tmp,path)                      # atomic swap
```

Why the guard: `open(path,'w')` truncates immediately; if the write then fails
(e.g. an encoding error), you get a 0-byte file. Encoding to bytes *first* and
swapping a temp file means a failure never corrupts the live file.

## Editing `SHOPPING` or JS/CSS

`SHOPPING` is a plain `const` in the JS (not inside `DATA`) precisely so its
emoji-heavy tier titles never go through `json.dumps`. Edit it (and any
JS/CSS/tab/view code) with `str_replace` on exact unique strings, or a Python
regex/insert that writes real characters. For structural JS refactors, brace-
match to extract a whole function reliably rather than fragile line matching.

## The emoji gotcha (important)

Never write an astral emoji as a **surrogate pair** in Python source
(`"\ud83c\udf9f"`) — Python keeps them as lone surrogates and `.encode('utf-8')`
throws "surrogates not allowed", which (without the encode-first guard) truncates
the file. Use the real character (`"🎟️"`) or a full code point (`"\U0001F39F\uFE0F"`).
BMP symbols like `⭐` (U+2B50) are fine either way.

## Reordering a walking tour into a loop

1. Read `references/…` routing rule. Decide circle vs furthest-first from the
   hotel location.
2. Rebuild `tour.stops` in geographic order (core stops, then `(optional)` ones,
   with `transit:true` connectors between legs).
3. Fix the `schedule` so its times follow the same path — no backtracking.
4. Update `tour.intro` to state the loop logic.
5. Reordering is safe for `seen:` keys only if the trip hasn't started; otherwise
   note that indices shift.

## Swapping two days' content

Move only the content fields (`headline, subtitle, overview, weather, walking,
transport, schedule, tour, callouts, eat, foods, good`); keep `n, weekday, date,
iso, short` with the slot. Then fix any weekday-specific notes (markets that run
Wed/Sat, Sunday shop closures) to match the day's new weekday.

## Validate every change (headless)

Run the app in headless Chromium and assert no console/page errors, plus a
targeted check for what you changed:

```js
const {chromium}=require('playwright');
(async()=>{const b=await chromium.launch();const p=await b.newPage();const e=[];
  p.on('console',m=>{if(m.type()==='error')e.push(m.text())});
  p.on('pageerror',x=>e.push(String(x)));
  await p.goto('file://'+process.cwd()+'/index.html');await p.waitForTimeout(400);
  for(const t of ['Today','Days','Places','Lists','Info']){await p.click('nav >> text='+t).catch(()=>{});await p.waitForTimeout(80);}
  await p.click('nav >> text=Days');await p.locator('.daycard').nth(N).click();
  // assert your new stop / callout / order renders …
  console.log('errors:', e.length?e:'NONE');
  await b.close();})();
```

Also good to check: stop numbers still read 1..K (transit rows unnumbered);
"Open in Maps" count matches real stops; Seen toggle persists across `reload()`;
shopping switch + tap-for-details works. Screenshot at 390px width to eyeball the
nav and any segmented control when you add lists/tabs.

## Deliver

`present_files` the `index.html` first, then the README. Keep the post-amble
short. Always remind: the file in outputs is current, but the **GitHub Pages copy
lags** — the user must re-upload `index.html` to push changes to the group.

## Deploy / share (tell the user)

- **GitHub Pages**: create a repo, upload `index.html`, enable Pages → link at
  `https://<user>.github.io/<repo>/`. Case-sensitive path; may 404 for a minute.
- **Offline**: open the link once, then **Add to Home Screen** — it caches and
  works with no signal (useful crossing borders).
- **Share**: one QR code of the Pages link for the whole group.
- Data (itinerary + leg colors) is editable near the top of `index.html`.
- Checkmarks/notes are per-device, not synced between travelers.
