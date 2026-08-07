# Scroll-Triggered Number Count-Up — Reference

Extracted from the Bosphorus scrollytelling project. This is the mechanism that makes the big
percentage stat (`−26.8%` → `+30.2%` → `+77.8%`) count up from zero each time a new step scrolls
into view.

## Where it lives in this project

| Piece | File | Lines |
|---|---|---|
| Easing function | [js/script.js:657-663](js/script.js#L657-L663) | `easeInOutCubic` |
| **The count-up animation** | [js/script.js:932-963](js/script.js#L932-L963) | `animateStatNumber` |
| Step activation (calls it) | [js/script.js:969-982](js/script.js#L969-L982) | `activateDataStep` |
| Scroll trigger | [js/script.js:984-1001](js/script.js#L984-L1001) | `IntersectionObserver` |
| Chart value morph (same trick, applied to chart data) | [js/script.js:870-926](js/script.js#L870-L926) | `animateChartToMetric` |
| Step markup + target numbers | [index.html:183-288](index.html#L183-L288) | `data-change` attributes |
| Sticky graphic / tall steps layout | [css/styles.css:270-427](css/styles.css#L270-L427) | `.data-scrolly`, `.data-step` |

## How it works — four parts

1. **HTML** stores the target number on each step as a `data-change` attribute, and holds a `<span>`
   whose text gets overwritten frame by frame.
2. **CSS** makes the graphic column `position: sticky` and each text step tall (`min-height: 82vh`),
   so scrolling the text past the pinned graphic is what drives the story.
3. **IntersectionObserver** fires when a step crosses 58% visibility and marks it active.
4. **requestAnimationFrame loop** interpolates `0 → target` over 650ms with cubic easing, writing
   `textContent` each frame.

No library. No scroll listener. No D3 needed for this part.

---

## 1. HTML

Each step is a normal block of text with the target value in a data attribute:

```html
<div class="data-scrolly__steps">
  <article
    class="data-step is-active"
    data-metric="vessels"
    data-title="Fewer vessels crossed the Bosphorus."
    data-subtitle="Annual vessel traffic fell 26.8% between 2006 and 2025."
    data-label="VESSEL COUNT"
    data-change="-26.8"
  >
    <p class="step__number">01</p>
    <h2>Traffic declined.</h2>
    <p>Nearly 55,000 vessels crossed the Bosphorus in 2006…</p>

    <div class="data-step__change">
      <span class="data-step__arrow" aria-hidden="true">↓</span>
      <span class="data-step__stat">−26.8%</span>   <!-- ← this text is animated -->
      <span class="data-step__caption">fewer vessels</span>
    </div>
  </article>

  <!-- repeat with data-change="30.2", "77.8", … -->
</div>
```

The hard-coded text inside `.data-step__stat` is the fallback for no-JS / pre-animation; the script
overwrites it immediately.

## 2. CSS (the scrollytelling scaffold)

```css
.data-scrolly {
  display: grid;
  grid-template-columns: minmax(0, 1.32fr) minmax(360px, 0.68fr);
  gap: 5vw;
}

/* Left column pins while the right column scrolls */
.data-scrolly__graphic {
  position: sticky;
  top: 0;
  height: 100svh;
  min-width: 0;               /* prevents grid blowout with SVG children */
  display: flex;
  flex-direction: column;
  justify-content: center;
}

/* Half-viewport padding so step 1 can activate and step N can stay active at the end */
.data-scrolly__steps { padding: 34vh 0; }

.data-step {
  min-height: 82vh;           /* one step ≈ one screen of scrolling */
  display: flex;
  flex-direction: column;
  justify-content: center;
  opacity: 0.2;
  transform: translateY(18px);
  transition: opacity 350ms ease, transform 450ms ease;
}

.data-step.is-active {
  opacity: 1;
  transform: translateY(0);
}

.data-step__stat {
  font-family: "Source Serif 4", serif;
  font-size: clamp(3.6rem, 4.6vw, 5.2rem);
  font-weight: 700;
  line-height: 0.88;
  letter-spacing: -0.045em;
  color: var(--accent);
}
```

> **Tip:** give the stat element `font-variant-numeric: tabular-nums;` if the number jitters
> horizontally while counting. This project uses a serif face where it's not noticeable, but most
> sans faces will wobble.

## 3. JavaScript — the animation itself

### Easing

```js
function easeInOutCubic(progress) {
  if (progress < 0.5) {
    return 4 * progress * progress * progress;
  }
  return 1 - Math.pow(-2 * progress + 2, 3) / 2;
}
```

### The count-up (verbatim from the project)

```js
function animateStatNumber(step) {
  const statElement = step.querySelector(".data-step__stat");

  if (!statElement) {
    return;
  }

  const targetValue = Number(step.dataset.change);
  const duration = 650;
  const startTime = performance.now();

  function animateNumber(now) {
    const elapsed = now - startTime;
    const progress = Math.min(elapsed / duration, 1);
    const easedProgress = easeInOutCubic(progress);

    const currentValue = targetValue * easedProgress;
    const sign = targetValue > 0 ? "+" : "−";

    statElement.textContent =
      `${sign}${Math.abs(currentValue).toFixed(1)}%`;

    if (progress < 1) {
      requestAnimationFrame(animateNumber);
    } else {
      // Snap to the exact target so rounding never leaves it at 26.9
      statElement.textContent =
        `${sign}${Math.abs(targetValue).toFixed(1)}%`;
    }
  }

  requestAnimationFrame(animateNumber);
}
```

**Key ideas:**

- It counts `0 → target`, always from zero, re-running every time the step re-enters view.
- `Math.abs()` + a separate `sign` variable means the minus sign never flickers between `-0.0` and
  `−0.1`. The sign character is U+2212 (`−`, true minus), not a hyphen.
- `.toFixed(1)` fixes the decimal count so the string width stays stable.
- The `else` branch snaps to the exact value — floating point at `progress === 1` can otherwise land
  a hair short.

### Wiring it to scroll

```js
const dataSteps = document.querySelectorAll(".data-step");

function activateDataStep(step) {
  dataSteps.forEach((item) => item.classList.remove("is-active"));
  step.classList.add("is-active");

  // Swap the sticky graphic's copy from the step's data attributes
  chartTitle.textContent = step.dataset.title;
  chartSubtitle.textContent = step.dataset.subtitle;
  chartMetricLabel.textContent = step.dataset.label;

  animateChartToMetric(step.dataset.metric);   // optional — see §4
  animateStatNumber(step);                     // ← the number count-up
}

const dataObserver = new IntersectionObserver(
  (entries) => {
    entries.forEach((entry) => {
      if (!entry.isIntersecting) {
        return;
      }
      activateDataStep(entry.target);
    });
  },
  {
    threshold: 0.58   // step must be 58% visible before it takes over
  }
);

dataSteps.forEach((step) => dataObserver.observe(step));
```

`threshold: 0.58` is the tuning knob. Lower = steps activate earlier / feel twitchier on fast
scrolls; higher = later, and with tall steps a value near 1 may never fire on short viewports.

---

## 4. Bonus: the same trick applied to chart values

`animateChartToMetric` is the identical eased-lerp pattern, but instead of one number it interpolates
an entire array of data points and redraws the chart each frame. Useful if your target project also
morphs a chart between steps.

```js
function animateChartToMetric(metric) {
  const newData = calculateIndexedData(metric);

  // First run: nothing to morph from
  if (!currentIndexedData.length) {
    currentIndexedData = newData;
    drawChart(newData, metric, true);
    currentMetric = metric;
    return;
  }

  // Cancel any in-flight morph so two loops don't fight
  if (chartAnimationFrame) {
    cancelAnimationFrame(chartAnimationFrame);
  }

  const startData = currentIndexedData.map((row) => ({ ...row }));
  const duration = 700;
  const startTime = performance.now();

  function animateFrame(now) {
    const elapsed = now - startTime;
    const progress = Math.min(elapsed / duration, 1);
    const easedProgress = easeInOutCubic(progress);

    const interpolatedData = newData.map((row, index) => {
      const startValue = startData[index]?.indexedValue ?? 100;

      return {
        year: row.year,
        rawValue: row.rawValue,
        indexedValue: startValue + (row.indexedValue - startValue) * easedProgress
      };
    });

    drawChart(interpolatedData, metric, false);   // redraw, no line-draw animation

    if (progress < 1) {
      chartAnimationFrame = requestAnimationFrame(animateFrame);
    } else {
      currentIndexedData = newData;
      currentMetric = metric;
      drawChart(newData, metric, true);
    }
  }

  chartAnimationFrame = requestAnimationFrame(animateFrame);
}
```

The lerp line is the whole idea:

```js
current = start + (end - start) * eased(progress)
```

Note it also indexes every metric to `100` at the first year ([js/script.js:625-635](js/script.js#L625-L635)),
which is what makes three different units share one y-axis and morph smoothly.

The chart's endpoint labels are formatted per metric ([js/script.js:547-573](js/script.js#L547-L573)):

```js
const metricSettings = {
  vessels:           { label: "VESSEL COUNT",               format: (v) => Math.round(v).toLocaleString("en-US") },
  gross_tonnage:     { label: "TOTAL GROSS TONNAGE",        format: (v) => `${Math.round(v / 1e6).toLocaleString("en-US")} million` },
  avg_gt_per_vessel: { label: "AVERAGE TONNAGE PER VESSEL", format: (v) => Math.round(v).toLocaleString("en-US") }
};
```

---

## 5. Drop-in generic version

Reusable and slightly hardened over the original — cancels stale loops, respects reduced motion, and
reads formatting from data attributes so it works for counts, currency and percentages alike.

```html
<article class="step" data-value="-26.8" data-decimals="1" data-prefix="" data-suffix="%">
  <span class="step__stat">−26.8%</span>
</article>
```

```js
const REDUCED_MOTION = window.matchMedia("(prefers-reduced-motion: reduce)").matches;
const activeFrames = new WeakMap();   // one in-flight rAF id per element

function easeInOutCubic(t) {
  return t < 0.5 ? 4 * t * t * t : 1 - Math.pow(-2 * t + 2, 3) / 2;
}

function countUp(step, { duration = 650 } = {}) {
  const el = step.querySelector(".step__stat");
  if (!el) return;

  const target   = Number(step.dataset.value);
  const decimals = Number(step.dataset.decimals ?? 0);
  const prefix   = step.dataset.prefix ?? "";
  const suffix   = step.dataset.suffix ?? "";
  const sign     = target < 0 ? "−" : (step.dataset.showPlus === "false" ? "" : "+");

  const render = (value) =>
    `${sign}${prefix}${Math.abs(value).toFixed(decimals)}${suffix}`;

  // Kill any loop still running on this element (fast scroll / re-entry)
  if (activeFrames.has(el)) cancelAnimationFrame(activeFrames.get(el));

  if (REDUCED_MOTION) {
    el.textContent = render(target);
    return;
  }

  const startTime = performance.now();

  function frame(now) {
    const progress = Math.min((now - startTime) / duration, 1);
    el.textContent = render(target * easeInOutCubic(progress));

    if (progress < 1) {
      activeFrames.set(el, requestAnimationFrame(frame));
    } else {
      el.textContent = render(target);   // snap to exact
      activeFrames.delete(el);
    }
  }

  activeFrames.set(el, requestAnimationFrame(frame));
}

const steps = document.querySelectorAll(".step");

const observer = new IntersectionObserver(
  (entries) => {
    entries.forEach((entry) => {
      if (!entry.isIntersecting) return;

      steps.forEach((s) => s.classList.remove("is-active"));
      entry.target.classList.add("is-active");

      countUp(entry.target);
      // …swap your sticky graphic here using entry.target.dataset
    });
  },
  { threshold: 0.58 }
);

steps.forEach((step) => observer.observe(step));
```

Add `observer.unobserve(entry.target)` inside the callback if you want each number to count up only
once instead of on every re-entry.

---

## Integration checklist

- [ ] Sticky graphic column needs a **non-sticky, scrolling sibling** — sticky does nothing if the
      parent isn't tall enough. Grid or flex parent, graphic gets `position: sticky; top: 0`.
- [ ] Every step needs real height (`min-height: 80vh`) or several fire at once.
- [ ] Pad the steps container top and bottom (`padding: 34vh 0`) so the first and last steps can
      actually reach the threshold.
- [ ] Don't put `overflow: hidden` on any ancestor of the sticky element — it silently breaks sticky.
- [ ] Fix the decimal count with `.toFixed(n)` and consider `font-variant-numeric: tabular-nums`.
- [ ] Cancel the previous `requestAnimationFrame` before starting a new one (the original
      `animateStatNumber` doesn't; the drop-in version above does).
- [ ] Ship a sensible static value in the HTML so the number is readable before JS runs.
- [ ] Mobile: under `max-width: 800px` this project switches the grid to `display: block` and shrinks
      the still-sticky graphic to `62svh` with an opaque background, so the text steps scroll
      underneath it — see [css/styles.css:660-724](css/styles.css#L660-L724).
- [ ] `prefers-reduced-motion` is handled in CSS for the step fades
      ([css/styles.css:761-778](css/styles.css#L761-L778)) but **not** for the JS count-up — the
      drop-in version above adds that guard.
```
