# Style system — the map sets the rules

**The problem this solves.** `index.html` was assembled from three sources: Aaron's
scrollytelling template, a scroll-scrubbed spending counter, and a chart migrated out
of `index3.html`. Each arrived with its own type, its own margins, and its own reds.
Nothing was wrong on its own; together they read as three pages stapled at the spine.

**The rule that fixes it.** The map is the largest, most memorable thing on the page,
and it is the one element whose colors were *not* chosen — they came out of
OpenMapTiles. So the map is not styled to match the page. **The page is styled to
match the map.** Every color, and the logic of where each color is allowed to go,
is derived from pixels in `images/anno1.png` and `images/anno3.png`.

---

## The process — eight steps, in order

Do them in this order. Color is step 3, not step 1, and layout is decided before
type, because a measure you have not fixed will make every type size look wrong.

### Step 1 — Sample the map. Don't guess its colors.

Never eyedrop by eye. Count pixels:

```bash
python3 -c "
from PIL import Image; from collections import Counter
im = Image.open('images/anno3.png').convert('RGB')
for col, n in Counter(im.getdata()).most_common(12):
    print('#%02x%02x%02x' % col, n)
"
```

What that returned, and what each color *does* in the map. This is the whole
vocabulary — the page may not invent an eighth color:

| Sampled | Hex | Its job in the map |
|---|---|---|
| land, in focus | `#EDCCE1` | the ordinary field — where the story happens |
| land, out of focus | `#B199A6` | same thing, pushed back |
| block / park fill | `#F0E2EC` | a quieter step of the field |
| streets | `#FFFFFF` | the gaps that make the field legible |
| water (anno1) | `#C5DADF` | not-the-subject |
| water (anno3, dimmed) | `#97A3A6` | not-the-subject, pushed back |
| restaurant dots | `#333333` | data, at rest |
| **highlight core** | `#99394C` | **data, being pointed at** |
| **highlight halo** | `#F693B3` | the "look here" ring around it |
| home icon | `#00C8B3` | you-are-here — used exactly once |

### Step 2 — Read the map's *grammar*, not just its colors.

The palette matters less than the logic underneath it. The map says four things:

1. **The field is pale.** Almost the entire frame is a low-chroma pink.
2. **Emphasis is one deep crimson, and it is rare.** Two dots out of thirty.
3. **Emphasis always ships with a soft halo** — the ring, not just the fill.
4. **Out-of-scope is desaturated, never hidden.** The water and the far boroughs
   are still there; they just stop competing.

Those four sentences are the design system. Everything below is bookkeeping.

### Step 3 — Declare tokens. One `:root` block, no hex anywhere else.

Two adjustments to the sampled values, both forced by contrast math, both keeping
the hue:

- `#99394C` → **`#A8264E`** so it clears 4.5:1 on white as text (6.88:1) *and*
  carries white labels on top of it (6.88:1).
- `#00C8B3` → **`#0E9382`** so the marker clears 3:1 as a mark (3.81:1).

```css
:root{
  /* ── color: sampled from images/anno1.png + anno3.png ───────────── */
  --field:       #EDCCE1;   /* map land, in focus */
  --field-soft:  #F8EEF4;   /* map block fill — the quietest tint */
  --dim:         #B199A6;   /* map land, out of focus */
  --water:       #C5DADF;   /* map water */
  --accent:      #A8264E;   /* map highlight core, raised to 4.5:1 */
  --accent-soft: #F693B3;   /* map highlight halo */
  --accent-tint: #F4DCE6;   /* accent at ~12% on white */
  --marker:      #0E9382;   /* map home icon, lowered to 3:1 */
  --ink:         #231C26;   /* map dot charcoal, warmed toward the field */
  --ink-muted:   #6B6070;
  --rule:        #E4D6E0;
  --surface:     #FFFFFF;

  /* ── type ───────────────────────────────────────────────────────── */
  --font-serif:  Georgia, "Times New Roman", serif;   /* everything you read */
  --font-sans:   -apple-system, ... , sans-serif;     /* everything you look up */

  /* ── measure & grid ─────────────────────────────────────────────── */
  --measure:     34rem;     /* ~68 characters at 18px Georgia */
  --content:     1180px;    /* full width for figures */
  --gutter:      1.5rem;
  --spine: max(var(--gutter),
               calc((100% - var(--content)) / 2 + var(--gutter)));

  /* ── spacing scale — 8px base, no value off this ladder ─────────── */
  --sp-xs: .5rem;  --sp-sm: 1rem;   --sp-md: 1.5rem;
  --sp-lg: 2.5rem; --sp-xl: 4rem;   --sp-2xl: 6rem;
}
```

**The enforcement rule:** after this step, a raw hex or a raw `px` margin anywhere
below `:root` is a bug. That includes the JavaScript — both charts read their colors
back out with `getComputedStyle(document.documentElement)`, so there is exactly one
place to change a color and it is this block.

### Step 4 — Fix the measure before touching type, and give the page one left edge.

The template shipped `h1, p { max-width: 50% }`. A percentage measure is the single
worst layout bug on the page: on a 1920px monitor it sets a 960px line — about 120
characters, roughly double the readable maximum — and on a laptop it sets 60. The
line length changes every time the window does, so no type size can be right.

Two rails replace it:

| Rail | Width | What sits on it |
|---|---|---|
| **prose** | `--measure` (34rem) | `h1`, every narrative `<p>` |
| **figure** | `--content` (1180px) | the spend stage, the chart row, the sticky map |

**Centering both is not enough** — and this is the part that is easy to get wrong.
Center a 34rem paragraph and a 1180px figure on the same page and their *centers*
agree while their *left edges* are 320px apart. The eye tracks left edges, so the
page still reads as two documents. The first attempt at this looked exactly as
broken as the version it replaced.

So the prose does not center. It hangs off the figure rail's inner edge:

```css
--spine: max(var(--gutter), calc((100% - var(--content)) / 2 + var(--gutter)));
h1, p { max-width: var(--measure); margin-left: var(--spine); }
```

The percentage resolves against the element's *containing block*, not the viewport,
so it stays correct when a classic scrollbar is present — `100vw` would drift by the
scrollbar's width. Two things this obliges:

- Any `--content`-wide box needs `box-sizing: border-box`, or its gutter lands
  outside the 1180px and its contents sit a gutter to the left of `--spine`.
- Sections whose children are laid out in `vw` (here, `#zoomBox`) do not get the
  spine on their container — put it on the text column only.

Result: the headline, every paragraph, the step cards, the spending readout, the
chart title, the chart legend and the plot's own left edge all start on one line.

### Step 5 — Match the chart's type to the page's type.

An SVG authored in a viewBox and scaled into a column renders its labels at
`size ÷ scale`. This chart is 980 units wide in a 672px track, so an 18-unit axis
label arrives on screen at **12px, beside 18px prose** — which is most of why a
migrated chart reads as a foreign embed no matter what colors it wears.

Convert the type tokens into viewBox units instead of writing sizes by eye:

```js
const WIDE_SCALE = 980 / 672;              // ≈ 1.46
const labelUnits = (px, compact) =>
  Math.round(px * (compact ? COMPACT_SCALE : WIDE_SCALE));

axisSize: labelUnits(14, compact),         // --fs-small
segSize:  labelUnits(12, compact),         // --fs-micro
```

Growing the labels shrinks the room they had, so the margins move with them: the
left margin only has to clear a two-digit tick plus its padding (74 → 46 units),
and the month and year rows need their baselines pushed down.

### Step 6 — One type scale, two families, assigned by job.

The map's own labels are set in a humanist sans. So the split is not decorative — it
is the same split the map already makes:

> **Serif is for text you read. Sans is for text you look up.**

Prose, headlines, and the step cards are Georgia. Axis ticks, legends, value labels,
eyebrows, dates, and captions are sans — the same job the map's street and place
labels do.

| Token | Size | Line height | Used for |
|---|---|---|---|
| `--fs-display` | `clamp(2.25rem, 5vw, 3.5rem)` | 1.08 | `h1` |
| `--fs-stat` | `clamp(3rem, 5.6vw, 4.75rem)` | 0.92 | the running total |
| `--fs-h2` | `clamp(1.35rem, 2.2vw, 1.75rem)` | 1.25 | section + chart titles |
| `--fs-lead` | `1.3125rem` (21px) | 1.55 | the nut graf only |
| `--fs-body` | `1.125rem` (18px) | 1.65 | **every** other paragraph |
| `--fs-small` | `.875rem` (14px) | 1.5 | captions, notes, chart labels |
| `--fs-micro` | `.75rem` (12px) | 1.4 | eyebrow, uppercase meta |

This collapses four accidental body sizes (19px global, 20px in steps, 17px in the
readout, 16px in the chart prose) into one.

### Step 7 — Assign color by role, from the map's grammar.

Not "what looks nice here" — *which of the map's four sentences is this element?*

| Role | Token | Rule |
|---|---|---|
| Reading text | `--ink` | never accent, never a series color |
| Secondary text | `--ink-muted` | captions, axis ticks, meta |
| **Emphasis** | `--accent` | the running total, the peak, the Double Dash series. Rare, by rule. |
| Emphasis surround | `--accent-soft` / `--accent-tint` | the halo equivalent — a soft ring or wash behind anything wearing `--accent` |
| Field / context | `--field`, `--field-soft` | section washes, the carried-in bars, quiet fills |
| Pushed back | `--dim`, `--rule` | hairlines, gridlines, out-of-scope |
| You-are-here | `--marker` | reserved. Used only where the map uses it. |

Two charts, two different color jobs — this distinction is why the palette does not
break:

- **Spending chart** = *sequential*. Base and cap are the same measure at two
  depths, so they take one hue, light → dark: `--field` → `--accent`.
- **Double Dash chart** = *categorical*. Solo and Double Dash are different things,
  so they take two slots: `#D4739F` and `--accent`, both inside the map's hue family.

Validate the categorical pair — do not reason about it:

```bash
node scripts/validate_palette.js "#A8264E,#D4739F" --mode light
#  [PASS] lightness band · chroma floor · CVD ΔE 18.6 deutan
#  [PASS] normal-vision ΔE 19.5 · contrast ≥ 3:1
```

Label contrast follows from the fills, and lands on the map's own habit of dark
marks on a pale field: **white numerals on the crimson segment (6.88:1), ink
numerals on the pink one (5.34:1).**

**Emphasis is never color alone — the halo is part of the move.** On the map the
highlighted dot gets a ring. In the Double Dash chart the peak column gets a wash of
`--accent-tint` behind it. That band deliberately runs *past* the plot at both ends,
above the tallest bar and down through the month label: a band that stopped at the
baseline would be bar-shaped, in a chart where pale pink is already a series color.

### Step 8 — Verify. Six greps and a screenshot.

Every check below is a number or a grep, not an opinion. All six pass as of this
writing.

1. `grep -n '#[0-9a-fA-F]\{6\}' index.html index3.html` returns **only** lines inside
   `:root` (plus comments naming a value that was replaced).
2. No `max-width` expressed as a percentage.
3. Every `font-family` is `var(--font-serif)` or `var(--font-sans)`.
4. Every `font-size` is a `--fs-*` token — including the two charts', which reach
   them through `labelUnits()` and through CSS rules on the inline SVG (a CSS rule
   beats a presentation attribute, so `font-size="12"` in the markup comes out).
5. Every margin and gap is a `--sp-*` token (scroll-track heights excepted — those
   are timing, not spacing).
6. `--accent` appears on ≤ 3 kinds of element. If it is on more, it has stopped
   meaning "look here."

Then **render it and look at it.** None of the six would have caught the two real
problems in this pass — prose centered against left-aligned figures, and chart labels
a third smaller than the prose. Measuring the left edge of every rail is worth the
thirty seconds:

```js
['h1','p.lede','.spend-readout','.scrolly-overlay','#step1','.chart-figure']
  .forEach(s => console.log(s, document.querySelector(s).getBoundingClientRect().left));
// → 154 for all six
```

---

## Inventory — what was inconsistent, and what it became

| Was | Where | Now |
|---|---|---|
| 4 body sizes (19 / 20 / 17 / 16px) | global `p`, `.step p`, readout, chart prose | one `--fs-body` |
| 3 type systems (default serif, sans, Georgia) | h1 / p / chart | serif for prose, sans for labels |
| `max-width: 50%` | `h1`, `p` | `--measure` on `--spine` |
| prose centered, figures left-aligned | whole page | one left edge, `--spine` |
| chart labels at ~12px beside 18px prose | both charts | `labelUnits()` off the type scale |
| 3 unrelated reds (`#ef2a39`, `#FF1F8E`, `palevioletred`) | counter, chart, modal | one `--accent`, from the map |
| `#7A86FF` periwinkle | chart's solo series | `#D4739F`, the map's pink family |
| `border: 3px solid red` | `.step` | debug artifact — removed |
| `background-color` declared twice | `.step` | one declaration |
| `border: black` (invalid) | `.step p` | removed |
| centered body text | `.step p` | left-aligned, in a card with a crimson edge |
| button flowing inline after the copy | `.modal` | `display: block` |
| ad-hoc margins (4rem/6rem/2rem/25rem) | everywhere | `--sp-*` ladder |
| `!important` + `vw` geometry | `#zoomBox` | `!important` dropped; geometry kept — it is hand-tuned to the artboards |

`index3.html` — the standalone preview the chart was migrated from — carries the same
`:root` block and the same chart script, so the two files cannot disagree. It is a
copy, not an import; change `index.html` and copy the block down.

## Three things deliberately left alone

- **`#zoomBox`'s `44vw` / `53vw` framing.** It is tuned to the Illustrator artboard,
  not to a grid. Moving it to the token scale would re-crop the map.
- **Scroll-track heights** (`460vh`, `25rem` between steps). These are *duration*,
  not spacing — they control how long the year takes to scroll past. They do not
  belong on the spacing ladder.
- **The step column is 26rem, not `--measure`.** It shares its row with the map,
  whose left edge is set in `vw`. At 34rem the cards run under the map from about
  1400px up. The measure is a maximum, not a quota.

## When you add the next section

1. Wrap prose in the measure rail; wrap figures in the content rail.
2. Pick its type from the scale. Do not add a size.
3. Ask which of the map's four sentences the new element is, and take that token.
4. If it needs a color that is not in `:root`, that is a signal the section is
   fighting the story — not a signal to add a token.
