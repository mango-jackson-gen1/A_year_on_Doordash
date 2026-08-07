# project3-demo2-images — one page, merged

[index.html](index.html) is now the whole story. The stacked column chart that lived in
`index3.html` has been migrated into it as the final section, and every conflict between the two
files has been resolved.

**`index3.html` is safe to delete.** Nothing in the project references it, and all of its
functional content is now in index.html — verified below.

| | |
|---|---|
| Page | [index.html](index.html) — h1 → scroll-scrubbed spend counter → photoDiv map sequence → Double Dash prose → **Double Dash stacked column chart** |
| Libraries | scrollama 3.0.4, d3 **7.3.0** (one copy), intersection-observer polyfill |
| Assets | `images/anno0-3.png`, `cumulative-123orders.csv` |

---

## 1. The data reconciles — 105 and 86 are the same year

The two numbers now appear on the same page, so this stopped being a footnote and became
mandatory.

```
86 deliveries + 19 Double Dash second stores = 105 store-level orders   ✓
```

The spend counter counts receipts (105 orders, `$4,345.83`). The chart counts drop-offs (86
deliveries, 19 of them Double Dash). Verified against both datasets:

- `SERIES`: **105** entries, last pair `[390, 4345.83]` — matches the hardcoded "of 105 delivery
  orders" at [index.html:302](index.html#L302).
- `orderData`: 13 months, solo `67` + double `19` = **86**.
- `MONTHS` cumulative deltas sum to `$4,345.83` exactly.
- **March 2026 is the peak in both** — `$970.12`, and 7 of 16 orders Double Dash.

Because a reader now sees 105 and 86 within one scroll, the chart's source note was extended to
close the gap explicitly ([index.html:517-523](index.html#L517-L523)): *"That is why this chart says
86 deliveries while the running total above counts 105 orders: 86 drop-offs plus the 19 second
stores a Double Dash adds."* This is the only copy change made during the migration.

---

## 2. Conflicts found and fixed

| # | Conflict | Resolution |
|---|---|---|
| 1 | **Global `p { max-width: 50%; font-family: sans-serif; font-size: 19px }`** at [index.html:40-45](index.html#L40-L45) is unscoped. It beats inheritance from `.story-text`, and `.story-text p` never declared `max-width` — dropped in as-is, the chart's prose column would have silently halved and reverted to sans-serif. | Two declarations added to `.story-text p`, marked `HOST OVERRIDE` in the source: `font-family: Georgia, serif` and `max-width: none`. `.story-note` inherits both through the same selector. The global rule itself was left alone so the template's own prose is untouched. |
| 2 | **Two d3 versions** — the page had 7.3.0, index3 loaded 7.8.5. | The 7.8.5 `<script>` tag was **not** migrated. Every API the chart uses (`selection.join`, `d3.groups`, `d3.sum`, `scaleBand`, `axisLeft`, `easeCubicInOut`) ships in 7.3.0. Confirmed working in the browser. |
| 3 | **Duplicate closing paragraphs.** index3 carried its own copy of the two template paragraphs that also existed in index.html. | The copy was dropped, along with its `.page-prose` styles. Both template paragraphs have since been **replaced entirely** by the Double Dash prose at [index.html:466-470](index.html#L466-L470) — no template placeholder text remains on the page. |
| 4 | **index3's body-level preview chrome** — `body { font-family: Georgia; padding: 4rem 1.5rem; min-height: 100vh; background: #fff }` plus `html, body { margin: 0; padding: 0 }`. index3 labelled this "PAGE CHROME — preview only." Migrated, it would have restyled the entire template. | **Not migrated.** Replaced by a scoped `.story-wrap { max-width: 1180px; margin: 6rem auto 4rem; padding: 0 1.5rem }`. This also retires the ~128px of dead scroll that `min-height: 100vh` without `border-box` used to add. |
| 5 | **Generic class name `.wrap`.** | Renamed `.story-wrap` — too generic to leave sitting in a shared stylesheet. |
| 6 | **`@media (max-width: 860px)`** | Rewritten as `@media only screen and (max-width: 860px)` to match the file's existing house style. |

### Conflicts accepted, not fixed

| # | Conflict | Why it stands |
|---|---|---|
| 7 | **Type split** — the page is sans-serif 19px; the chart section is Georgia. | index.html was *already* mixed: `h1` sets no `font-family`, so it renders in the browser's default serif while `p` is sans-serif. The chart's Georgia is load-bearing — its SVG labels, legend and annotation are all sized against it. Scoped to `.story`, it reads as a distinct final module. **To flip it to sans-serif:** change `font-family` on `.chart-figure` and `.story-text`, plus the `"Georgia"` attributes in the chart script. |
| 8 | **Palette split** — `#ef2a39` red above, `#7A86FF`/`#FF1F8E` below. | The two mean different things: red is "money spent" (one series), the blue/pink pair is "which kind of order" (a CVD-validated categorical pair — ΔE 18.3 protan). Collapsing them to one color would imply a relationship that isn't there. Noted in the source at [index.html:900-903](index.html#L900-L903). |
| 9 | **Chart sits left, prose right** — while the paragraph directly above it says the pattern "works best… with prose on the left and changing annotations on that base map on the right." | The paragraph is template instruction copy about photoDivs, not about this chart. Swap `grid-template-columns` and the DOM order if you want them to agree. |

---

## 2a. The lede

The template line ("This template will help cheat a kind of scrolly telling map…") was replaced with
the GDPR/digital-footprint intro at [index.html:432](index.html#L432). **No template placeholder copy
remains anywhere on the page.**

### The fragment was fixed

The first draft's second sentence had no main verb ("With the European Union's GDPR — how many
internet users have reached out…"). It was flagged rather than silently rewritten, and the revised
copy supplied fixes it: *"With the European Union's GDPR in place, I wonder how many internet users
have actually reached out to request a copy of their data."* The aside in the third sentence now
sits inside paired em dashes. Nothing was changed on our side beyond entity-encoding.

### Position: nut graf, not opener

The paragraph sits **after** the scroll-scrubbed spend counter, not before it, so "here are some of
the cumulative findings" arrives once the reader has already seen `$4,345.83`. That is a defensible
hook-then-nut-graf structure and was left alone; move it above `<section class="spend-scrolly">` if
you want it to open the piece instead.

---

## 2b. Fact-check of the Double Dash prose

The three paragraphs at [index.html:466-470](index.html#L466-L470) sit directly above a chart that
states its own numbers, so every figure in them has to survive the comparison.

### Changed: "nearly a third" → "more than a fifth"

| | |
|---|---|
| As drafted | "Nearly a third of my total orders were Double Dash." |
| The data | 19 Double Dash of **86 deliveries = 22.1%**. Against store-level orders it is 19 of 105 = 18.1%. |
| "Nearly a third" would need | 29 of 86, or 35 of 105. |
| Likely origin of the slip | **19 / 67 = 28.4%** — dividing by *solo* orders rather than the total. |
| Why it could not stand | The chart three inches below says "19 of 86 deliveries — 22% of the total." The page would have contradicted itself on screen. |

Two words changed, sentence otherwise untouched. **To revert**, replace `More than a fifth` with
`Nearly a third` at [index.html:466](index.html#L466) — but the chart copy would then need to change
too.

### Flagged, not changed: $27 × 19 ≠ $486

The paragraph says add-ons averaged **$27** and totalled **$486** over the year. With 19 Double Dash
orders that product is **$513**. $486 implies either 18 add-ons at $27, or 19 averaging **$25.58**.

I have no per-order add-on data to settle which figure is right — `cumulative-123orders.csv` and the
`SERIES` array only carry running totals, not a store-level breakdown — so both numbers are left
exactly as written. Worth a pass against the receipts.

### Checks that passed

| Claim | Verdict |
|---|---|
| "two cream puffs at $13.75 apiece" | $27.50 — consistent with the $27 average ✓ |
| "the little extra… was two-thirds the size of the meal" | $27 / $40 = 67.5% ✓ |
| "my actual dinner that night averaged around $40" | Whole-year average per order is $41.39. Excluding $486 of add-ons, the other 86 orders average $44.88 — so $40 is a touch low but defensible ✓ |
| "$486 over the year" as a share of the total | 11.2% of $4,345.83 — unused in the copy, but available if you want it |

---

## 2c. The chart section

The chart's headline and body copy were replaced with the dessert-habit prose, the chart gained a
visible title, and the legend label was corrected.

### Chart title (new)

`<h2 id="chart-title" class="chart-title">Double Dash increasing trend</h2>`, above the SVG in the
figure column ([index.html:486-488](index.html#L486-L488)).

The claim it makes is **supported by the data**: 14 of the 19 Double Dash orders fall in the last six
months (74%), against 5 in the first seven. The date range it does not state is still on the page
twice — on the axis's year row, and in the source note.

Three knock-on changes this required:

- The title element took over `id="chart-title"` from the old visually-hidden `<p>`, which was
  deleted. `aria-labelledby` still resolves, now to the visible heading.
- `updateChartAccessibility()` no longer writes to `#chart-title` — that line would have blanked the
  visible title on every draw. It still generates the hidden `#chart-description`.
- `.story-text h2` was removed from the CSS (nothing matches it now) and replaced by `.chart-title`.

### Legend label

`"Solo order"` → `"Solo Order"`, in both the `SERIES` definition and the `segments` array, so the
swatch matches the title-case of "Double Dash". Confirmed rendering as `["Double Dash", "Solo Order"]`.

### Fact-check of the new prose

| Claim | Verdict |
|---|---|
| "frequency… increased significantly during the latter six months" | **Verified.** Feb–Jul 2026 holds 14 of the 19 Double Dash orders (74%); the first seven months hold 5. |
| "Lady M's Amedeo chocolate mille-feuille cake… 4 times, 7 pieces" | **Cannot verify.** No item-level data in the project — `cumulative-123orders.csv` and `SERIES` carry running totals only. |
| "Patseeri's Swiss cake roll… 5 times, 6 pieces" | **Cannot verify**, same reason. Worth checking the shop name against the receipts — the spelling did not resolve to a bakery I could confirm. |
| Internal consistency | 4 + 5 = 9 dessert trips of 19 Double Dash orders (47%) — plausible against "a habit". |

### Gridlines removed

`d3.axisLeft(...).tickSize(-(width - margin.left - margin.right))` → `.tickSize(0)`, and the two
`.call(g => g.selectAll(".tick line")…)` blocks that coloured and then re-hid them are gone.

The y-axis value labels (0, 4, 8, 12, 16) **do not move**, because d3 positions tick labels at
`Math.max(tickSizeInner, 0) + tickPadding` — with a negative `tickSize` that was already just
`tickPadding`. The hand-drawn baseline and y spine are separate `<line>` elements and are untouched:
5 tick elements remain in the DOM, all with zero length, and both axis lines still render.

### The source note was removed

Removed at request, together with the hairline rule above it (that rule was `.story-note`'s own
`border-top`, so deleting the paragraph took it with it). The now-dead `.story-text .story-note` CSS
was deleted too.

**The chart no longer carries a source line or the 86/105 counting rule.** The spend counter still
says 105 orders a few screens up while the chart says 86 — nothing on the page now explains the gap.
Flagged once and decided; noted here only so it is not rediscovered later as an oversight.

---

## 2d. Scrollytelling steps

The section was cut from four steps to **three**, and all `STEP ONE/TWO/THREE/FOUR` labels are gone.

| Step | Copy | Image | Pairing |
|---|---|---|---|
| 1 | 102 orders / 32 restaurants / East Village + Nomad-Flatiron | `anno1.png` — all restaurants labelled, no house | ✓ |
| 2 | "Some of my most visited restaurants are… within walking distance of my house" | `anno2.png` — adds the teal house | partial, see below |
| 3 | Dim Sum Sam 17x, Din Tai Fung 8x | `anno3.png` — adds the shaded area and puts **red halos on exactly those two** | ✓ strong |

**Why a step was dropped.** The old steps 1 and 2 showed the *same picture*: a pixel comparison put
old `anno0.png` and old `anno1.png` at 0.18% difference — resampling noise on label edges only — and
`anno0` was exported at 1266x949 against 2631x1972 for the rest. It was one frame twice, the first
at half resolution, so the "reveal" between those steps changed nothing on screen. That is exactly
the failure the template warns about: *always use the artboard, keep all save settings the same*.

**Image files were then renamed** (outside this session): old `anno0` → `anno1`, the full-resolution
duplicate deleted, `anno2`/`anno3` unchanged. The numbering now matches the three steps. Verified all
three decode in-browser and the sequence runs `anno1 → anno2 → anno3`.

**One resolution mismatch survives.** The surviving `anno1.png` is the 1266x949 export; `anno2.png`
and `anno3.png` are 2631x1972. At the ~758px display width it is fine on a 1x screen but will look
softer than the other two on a retina display. Re-export it from the same artboard at the same
settings to match.

### Text/image mismatch at step 2

Step 2 asserts "within walking distance of my house," but `anno2.png` only places the house among
the restaurants — there is no distance rendered on it. The shaded polygon that could carry that idea
appears in `anno3.png`, one beat later, and it spans from Din Tai Fung (~50th St) down to Somtum Der
in the East Village. Against the map's own 500 m scale bar that is roughly 4 km end to end, so the
polygon reads as the extent of all orders, not a walkshed.

Step 3 then splits the claim: of the two restaurants it highlights, **Dim Sum Sam is ~250 m from the
house** (a genuine walk) while **Din Tai Fung is ~1.5 km north** (not one). The copy survives this
because it is hedged — "*some* of my most visited" — but if you want the walking-distance point to
land visually, it needs an actual walkshed on `anno2.png`.

### Step 1 copy

- **The map backs the geography.** `anno1.png` shows two clear clusters with a visible gap between
  Breads Bakery and Tim Ho Wan — a lower group (Somtum Der, Hanoi House, Raku, 886, Crispy Burger,
  Tim Ho Wan, Little Cheeks) in the East Village, and a larger band above it. "Nomad/Flatiron" fits
  the bulk of that band, though its northern end (BCD Tofu House, Cho Dang Gol, bb.q Chicken) is
  really Koreatown/Herald Square.
- **"32 unique restaurants" vs. the map.** I count **29 labelled restaurants** on `anno1.png`. The
  missing three could be overlapping dots sharing a label, or sites outside the crop — but if the
  map is meant to show all of them, it is worth a recount.
- **A third order count now appears on the page.** The spend counter says **105** orders, this step
  says **102** Manhattan orders, and the chart says **86** deliveries. 102 only makes sense as a
  subset of the 105 store-level orders (it exceeds 86), so the framing is coherent — but the note
  that used to explain 86 vs 105 was removed last round, so nothing on the page reconciles any of
  the three now.
- Internally consistent with step 3: 17 + 8 = 25 orders from two restaurants leaves 77 across the
  other 30, about 2.6 each.

### Step 2 copy

Template copy replaced with the Manhattan/restaurant-frequency prose at
[index.html:460](index.html#L460). The `STEP THREE` label and the teal `<span>` went with it.

- **The image supports the text.** `anno2.png` labels both restaurants the new copy names — **Dim
  Sum Sam** and **Din Tai Fung** — so the annotation and the prose now point at the same thing.
- **"Dim Sum Sam about 17 times, Din Tai Fung 8 times" cannot be verified** from anything in the
  project. There is no restaurant-level data, only running totals. 17 + 8 = 25 of 105 orders, which
  is at least plausible.
- **The teal house is now unexplained until step 4.** It is still drawn on `anno2.png`, but step 3
  no longer introduces it; step 4's "within walking distance of my house" is the first mention.
- **All template step labels are now removed.** The final step's typos were corrected at the same
  time: "Manhanttan" → "Manhattan", "restaurnats" → "restaurants", and the doubled period.
- Possible lead on the unverified "Patseeri" in the chart prose: `anno2.png` labels a shop
  **"ando Pâtisserie"**. "Patseeri" may be a rendering of "Pâtisserie" — worth confirming against
  the receipts.

---

## 3. Pre-existing issues, untouched

None of these were introduced by the migration, and none are in its path.

| # | Issue | Where |
|---|---|---|
| 10 | `.step { border: 3px solid red }` — template debug styling still switched on, so every step box is outlined in red | [index.html:83](index.html#L83) |
| 11 | ~~Typos in STEP FOUR~~ — **fixed** when the step was renumbered to step 3 | [index.html:452](index.html#L452) |
| 12 | `105` hardcoded in markup while `SERIES.length` is the real source | [index.html:302](index.html#L302) |
| 13 | `<html>` has no `lang` attribute | [index.html:4](index.html#L4) |
| 13b | ~~No `<title>` element~~ — **fixed**: added `<title>A Year on DoorDash</title>` alongside the h1 rename | [index.html:7](index.html#L7) |
| 14 | `justify-content: center` declared twice in `.sticky-thing` | [index.html:71-73](index.html#L71-L73) |
| 15 | Mobile modal fires at ≤900px but the side-by-side breakpoint is 700px — a 700-900px window gets the desktop layout *and* the "you're on a phone" warning | [index.html:249](index.html#L249), [index.html:712](index.html#L712) |
| 16 | `Y_MAX = 16` is hardcoded and March's total is exactly 16 — any future month above 16 would draw outside the plot | chart script |

---

## 4. Verification

Served over `python3 -m http.server 8791` and driven in headless Chrome over the DevTools Protocol.

**Completeness of the migration**

- Chart JavaScript is **identical to index3's, character for character**, after stripping comments.
  All 45 functions and constants present.
- Every CSS selector accounted for: migrated, or dropped on purpose (`body`, `html`, `.wrap`,
  `.page-prose`, the 600px preview query).
- Confirmed index3's `body`/`html` rules did **not** leak in, and index.html still has exactly one
  global `p` rule.
- One d3 version loaded (7.3.0). No duplicate element IDs anywhere in the merged file (21 IDs).
- HTML re-parsed: no unclosed or mismatched tags.

**Rendered behavior — the migrated chart**

| Check | 1440px | 800px | 390px |
|---|---|---|---|
| Bars / segment labels / totals / legend | 21 / 21 / 13 / 2 | 21 / 21 / 13 / 2 | 21 / 21 / 13 / 2 |
| Peak annotation | 1 | 1 | 0 (suppressed when compact, by design) |
| `viewBox` | `0 0 980 520` | `0 0 980 520` | `0 0 360 380` (compact) |
| `.story` grid | 2 cols (667 / 417) | collapsed to 1 | collapsed to 1 |
| `.story-text p` max-width | `none` ✓ | `none` ✓ | `none` ✓ |
| `.story-text p` font | Georgia 16px ✓ | Georgia ✓ | Georgia ✓ |
| `.story-note` font / max-width | Georgia 13.6px / `none` ✓ | ✓ | ✓ |
| Horizontal overflow | none | none | none |

21 bars is correct: 26 possible segments (13 months × 2) minus the 5 months with zero Double Dash,
filtered out by `s.value > 0`.

**Rendered behavior — the sections that were already there**

- Spend counter scrubs to `$4,345.83`, `Jul 31, 2026`, `105` orders, track at 100%, 26 rects — at
  all three widths.
- photoDiv sequence swaps `anno0 → anno1 → anno2 → anno3` under real incremental scrolling.
- Console across all widths: one 404 for `/favicon.ico`. Nothing else. No exceptions.

**One false alarm during verification.** Jumping straight to `#step4` with `window.scrollTo` left
the photo on `anno0.png`, which looked like the migration had broken scrollama. It had not —
scrollama's IntersectionObserver does not fire on an instantaneous jump. Re-tested with gradual
120px scroll increments and all four steps fired in order.
