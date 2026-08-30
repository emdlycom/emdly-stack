---
name: design-review
owner: launifycorp
category: design
description: A design review is not a list of everything wrong with a page. Anyone can produce forty observations. The work is deciding which three matter, and showing why. This skill is built around that: evidenc...
version: v1
license: MIT.
updated: 2026-08-30
recommended: true
security_checked: true
url: https://emdly.com/skills/launifycorp/design-review
raw: https://emdly.com/raw/launifycorp/design-review.md
install: npx @emdly/cli add launifycorp/design-review
---

---
name: design-review
description: Audit the visual design, typography, accessibility and distinctiveness of a live website by driving a browser at it, then optionally fix what it finds in source. Use when asked to review the design of a site or page, critique a landing page, do a design or UI audit, check a page for accessibility and contrast problems, find out why a site looks generic or "AI-generated", check how a page holds up on mobile, or QA a redesign before it ships.
---

# Design review

A design review is not a list of everything wrong with a page. Anyone can produce forty observations. The work is deciding which three matter, and showing why. This skill is built around that: **evidence first, opinion second, one ranked list, one headline.**

Two rules govern everything below.

1. **Measure before you judge.** Do not form an opinion about contrast, spacing or size and then go looking for numbers to support it. Run the probes, read the output, then interpret.
2. **Separate Defect from Judgment.** A *Defect* violates a published standard you can cite. A *Judgment* is a craft call made on your own authority. Never let the second borrow the credibility of the first. Every finding is tagged one or the other.

## BROWSER

`BROWSER` means whatever automation the host provides. Map these four verbs onto it; if one is unavailable, say so in the report and skip the probes needing it rather than guessing at results.

`BROWSER.goto(url)` navigate · `BROWSER.viewport(w,h)` resize · `BROWSER.shot()` screenshot · `BROWSER.eval(js)` run JS in page context, return JSON

Probes are expressions for `BROWSER.eval`. They read only, except the focus probe (restores state) and the text-spacing probe (injects a style you then remove).

---

## Pass 0 — Blink

Before any probe, before DevTools, before scrolling. Load and screenshot at **390×844 (mobile) first**, then 1440×900. Mobile leads because cramped layouts fail there first, and 320 CSS px is the width WCAG actually names in SC 1.4.10 — the narrow case is the one with a standard behind it.

Then write **one sentence**: *what does this page appear to be, and who is it for?* Write it before you know anything else.

This sentence is data. It is the only proxy you get for a first-time visitor, and it is perishable — once you have measured the page you cannot un-see it. If the sentence is vague ("some kind of software product"), that vagueness is the most important finding on the page, and no contrast ratio outranks it. Record it verbatim. Also note, from the screenshots alone: what your eye lands on first, and whether that is the thing the page wants you to do.

---

## Pass 1 — Extract

Mechanical only. Collect numbers, form no opinions. Keep raw output; you will cite it in Pass 2.

### 1.1 Rendered type inventory

What the page actually renders, not what the stylesheet says.

```js
(() => {
  const seen = new Map();
  document.querySelectorAll('body *').forEach(el => {
    if (![...el.childNodes].some(n => n.nodeType === 3 && n.textContent.trim())) return;
    const s = getComputedStyle(el);
    if (s.display === 'none' || s.visibility === 'hidden') return;
    const px = parseFloat(s.fontSize);
    const lh = s.lineHeight === 'normal' ? null : parseFloat(s.lineHeight);
    const fam = s.fontFamily.split(',')[0].replace(/["']/g, '').trim();
    const key = [px, lh, s.fontWeight, fam, s.letterSpacing].join('|');
    const r = seen.get(key) || { fontPx: px, linePx: lh, ratio: lh ? +(lh / px).toFixed(2) : null,
      weight: s.fontWeight, family: fam, tracking: s.letterSpacing, count: 0, widestPx: 0, sample: '' };
    r.count++;
    r.widestPx = Math.max(r.widestPx, Math.round(el.getBoundingClientRect().width));
    if (!r.sample) r.sample = el.textContent.trim().slice(0, 60);
    seen.set(key, r);
  });
  const out = [...seen.values()].sort((a, b) => b.count - a.count);
  out.forEach(r => r.approxCPL = Math.round(r.widestPx / (r.fontPx * 0.5)));
  return { styleCount: out.length, styles: out.slice(0, 25) };
})()
```

`approxCPL` estimates characters per line (0.5em average advance is serviceable for Latin text but varies by face). A flag to look at, not a number to quote. A high `styleCount` — every heading a slightly different size — is itself a finding.

### 1.2 Contrast

Walks real text nodes, resolving effective background by climbing ancestors. It **declines to score** text over an image or a semi-transparent layer rather than guessing; those need a screenshot sample and your eye.

```js
(() => {
  const lum = ([r, g, b]) => {
    const f = c => (c /= 255) <= 0.03928 ? c / 12.92 : ((c + 0.055) / 1.055) ** 2.4;
    return 0.2126 * f(r) + 0.7152 * f(g) + 0.0722 * f(b);
  };
  const parse = s => (s.match(/[\d.]+/g) || []).map(Number);
  const ratio = (a, b) => { const [x, y] = [lum(a), lum(b)].sort((p, q) => q - p);
    return +((x + 0.05) / (y + 0.05)).toFixed(2); };
  const bgOf = el => {
    for (let n = el; n && n !== document.documentElement; n = n.parentElement) {
      const s = getComputedStyle(n);
      if (s.backgroundImage !== 'none') return { unmeasurable: 'background-image' };
      const c = parse(s.backgroundColor);
      if (c.length >= 3 && (c[3] === undefined || c[3] === 1)) return { rgb: c.slice(0, 3) };
      if (c[3] > 0 && c[3] < 1) return { unmeasurable: 'alpha background' };
    }
    return { rgb: [255, 255, 255] };
  };
  const out = [], skipped = [], done = new Set();
  const w = document.createTreeWalker(document.body, NodeFilter.SHOW_TEXT);
  for (let n; (n = w.nextNode());) {
    const txt = n.textContent.trim(); if (!txt) continue;
    const el = n.parentElement;
    if (!el || done.has(el) || /^(script|style|noscript)$/i.test(el.tagName)) continue;
    done.add(el);
    const s = getComputedStyle(el), box = el.getBoundingClientRect();
    if (s.visibility === 'hidden' || s.display === 'none' || !box.width || !box.height) continue;
    if (parseFloat(s.opacity) === 0) continue;
    const px = parseFloat(s.fontSize), wt = parseInt(s.fontWeight) || 400;
    const large = px >= 24 || (px >= 18.66 && wt >= 700);   // WCAG 18pt / 14pt bold
    const need = large ? 3 : 4.5;
    const bg = bgOf(el);
    if (bg.unmeasurable) { skipped.push({ text: txt.slice(0, 40), why: bg.unmeasurable }); continue; }
    const r = ratio(parse(s.color).slice(0, 3), bg.rgb);
    if (r < need) out.push({ text: txt.slice(0, 50), ratio: r, needs: need, fontPx: px, weight: wt,
      large, color: s.color, bg: `rgb(${bg.rgb.join(',')})`,
      sel: el.tagName.toLowerCase() + (typeof el.className === 'string' && el.className.trim()
        ? '.' + el.className.trim().split(/\s+/).slice(0, 2).join('.') : '') });
  }
  return { failCount: out.length, failures: out.sort((a, b) => a.ratio - b.ratio).slice(0, 30),
           unmeasurable: skipped.slice(0, 10) };
})()
```

### 1.3 Target size

```js
(() => {
  const sel = 'a[href],button,input,select,textarea,summary,[role=button],[role=link],[role=tab],[onclick],[tabindex]:not([tabindex="-1"])';
  const small = [];
  document.querySelectorAll(sel).forEach(el => {
    const s = getComputedStyle(el);
    if (s.display === 'none' || s.visibility === 'hidden') return;
    const r = el.getBoundingClientRect();
    if (!r.width || !r.height || (r.width >= 24 && r.height >= 24)) return;
    const p = el.parentElement;   // SC 2.5.8 "inline" exception: sits in a run of text
    const inline = !!(p && s.display.includes('inline')
      && p.textContent.trim().length > el.textContent.trim().length + 10);
    small.push({ tag: el.tagName.toLowerCase(), w: Math.round(r.width), h: Math.round(r.height),
      label: (el.innerText || el.value || el.getAttribute('aria-label') || '').trim().slice(0, 30),
      likelyInlineException: inline });
  });
  return { under24: small.filter(t => !t.likelyInlineException),
           inlineExempt: small.filter(t => t.likelyInlineException).length };
})()
```

The **spacing exception** in SC 2.5.8 rescues an undersized target if a 24px circle centred on it touches no other target. This probe does not test that — verify by hand before calling an undersized target a defect.

### 1.4 Focus visibility

Mutates focus, then restores it. Catches `outline: none` with nothing put back.

```js
(() => {
  const prev = document.activeElement;
  const sel = 'a[href],button,input,select,textarea,[tabindex]:not([tabindex="-1"])';
  const all = [...document.querySelectorAll(sel)], bad = [];
  all.slice(0, 120).forEach(el => {
    const b = getComputedStyle(el);
    const before = { sh: b.boxShadow, bc: b.borderColor, bg: b.backgroundColor };
    el.focus({ preventScroll: true });
    const a = getComputedStyle(el);
    const changed = (a.outlineStyle !== 'none' && parseFloat(a.outlineWidth) > 0)
      || a.boxShadow !== before.sh || a.borderColor !== before.bc || a.backgroundColor !== before.bg;
    if (!changed) bad.push({ tag: el.tagName.toLowerCase(), outline: a.outlineStyle,
      label: (el.innerText || el.getAttribute('aria-label') || '').trim().slice(0, 30) });
    el.blur();
  });
  if (prev && prev.focus) prev.focus({ preventScroll: true });
  return { noVisibleFocus: bad, checked: Math.min(120, all.length) };
})()
```

A detected change means *something* renders on focus, not that it is adequate — confirm with a screenshot of a focused control. Judging indicator thickness and contrast-change is SC 2.4.13, which is **AAA**; hold a page to it only if asked.

### 1.5 Reflow, zoom lock, text spacing

Set `BROWSER.viewport(320, 800)`, then:

```js
(() => {
  const de = document.documentElement, over = [];
  if (de.scrollWidth > de.clientWidth + 1) {
    document.querySelectorAll('body *').forEach(el => {
      const r = el.getBoundingClientRect();
      if (r.width && r.right > de.clientWidth + 1) over.push({ tag: el.tagName.toLowerCase(),
        cls: (typeof el.className === 'string' ? el.className : '').slice(0, 40),
        right: Math.round(r.right), w: Math.round(r.width) });
    });
  }
  const c = (document.querySelector('meta[name=viewport]') || {}).content || '';
  return { horizontalScroll: de.scrollWidth > de.clientWidth + 1, docWidth: de.scrollWidth,
    viewport: de.clientWidth, offenders: over.slice(0, 15), viewportMeta: c,
    zoomLocked: /user-scalable\s*=\s*(no|0)/i.test(c) || /maximum-scale\s*=\s*1(\.0)?\b/i.test(c) };
})()
```

Horizontal scroll at 320px fails SC 1.4.10; `zoomLocked` blocks the 200% resize SC 1.4.4 requires. For **SC 1.4.12** inject the exact override values the criterion names, screenshot, then remove `#__ts` and confirm nothing clipped or overlapped:

```js
document.head.insertAdjacentHTML('beforeend', `<style id="__ts">
  * { line-height: 1.5 !important; letter-spacing: .12em !important; word-spacing: .16em !important; }
  p { margin-bottom: 2em !important; }</style>`)
```

### 1.6 System coherence

Not a standard — a count of how many distinct decisions the page makes where one would do.

```js
(() => {
  const bag = { color: {}, bg: {}, radius: {}, shadow: {}, space: {}, family: {}, dur: {} };
  const skip = new Set(['none', 'normal', '0', '0px', '0s', 'rgba(0, 0, 0, 0)', 'auto']);
  const add = (k, v) => { if (v && !skip.has(v)) bag[k][v] = (bag[k][v] || 0) + 1; };
  document.querySelectorAll('body *').forEach(el => {
    const s = getComputedStyle(el);
    if (s.display === 'none') return;
    add('color', s.color); add('bg', s.backgroundColor); add('radius', s.borderRadius);
    add('shadow', s.boxShadow); add('dur', s.transitionDuration);
    add('family', s.fontFamily.split(',')[0].replace(/["']/g, ''));
    [s.gap, s.paddingTop, s.paddingLeft, s.marginBottom].forEach(v => add('space', v));
  });
  return Object.fromEntries(Object.entries(bag).map(([k, v]) => [k, {
    distinct: Object.keys(v).length,
    top: Object.entries(v).sort((a, b) => b[1] - a[1]).slice(0, 8) }]));
})()
```

Read the **spacing scale** especially. Values all multiples of one base (4 or 8px) indicate a system; a spread like 13/17/22/31px means each component was spaced by hand, and the page reads as slightly loose everywhere without any single element looking wrong.

### 1.7 Distinctiveness signals

The convergence of generated and templated sites on one look is documented, and its origin is unusually traceable: Tailwind's author Adam Wathan publicly apologised "for making every button in Tailwind UI `bg-indigo-500` five years ago, leading to every AI generated UI on earth also being indigo" (Aug 2025). The mechanism — a default that saturated tutorials, then training data, then output that becomes training data — is set out by Alan West on DEV and in *Why Your AI Keeps Building the Same Purple Gradient Website* (prg.sh); designer Yusuf at 925Studios calls the indigo-to-purple gradient "the single loudest AI tell in 2026".

These are **signals, never verdicts**. A purple gradient on a brand whose logo is purple is correct. Report a cluster, never a single hit.

```js
(() => {
  const hits = [], push = (signal, detail) => hits.push({ signal, detail });
  const all = [...document.querySelectorAll('body *')];
  // indigo/violet/purple: blue clearly dominant, red between green and blue.
  // Catches tailwind indigo-500 rgb(99,102,241), violet-500 (139,92,246), purple-500 (168,85,247).
  const isPurple = ([r, g, b]) => b > 180 && b - g > 70 && r >= g - 15 && b - r > 30;
  const anyPurple = str => (str.match(/rgba?\([^)]+\)/g) || [])
    .some(m => isPurple((m.match(/[\d.]+/g) || []).map(Number)));
  let grad = 0, purp = 0;
  all.forEach(el => {
    const s = getComputedStyle(el);
    if (/gradient/.test(s.backgroundImage)) { grad++; if (anyPurple(s.backgroundImage)) purp++; }
    if (anyPurple(s.backgroundColor)) purp++;
  });
  if (purp) push('indigo/violet fill', `${purp} elements`);
  if (grad) push('gradient fills', `${grad} elements`);
  // Basis: the families Google Fonts serves most, in served-volume order,
  // plus the Tailwind and OS default sans stacks. A body face that is the single
  // most-served typeface on the web is a decision not yet made, not a choice.
  const STOCK = /^["']?(Roboto|Open Sans|Noto Sans|Lato|Montserrat|Poppins|Inter|ui-sans-serif|system-ui|-apple-system)/i;
  const fam = getComputedStyle(document.body).fontFamily.trim();
  if (STOCK.test(fam)) push('most-served default face', fam);
  all.forEach(el => {                       // three-up card row
    const s = getComputedStyle(el);
    if (!/flex|grid/.test(s.display)) return;
    const w = [...el.children].map(c => Math.round(c.getBoundingClientRect().width));
    if (w.length === 3 && w[0] > 120 && new Set(w).size === 1)
      push('three-up card row', `${el.tagName.toLowerCase()} · 3 × ${w[0]}px`);
  });
  const radii = new Set(all.map(el => getComputedStyle(el).borderRadius).filter(r => r !== '0px'));
  if (radii.size === 1) push('single radius everywhere', [...radii][0]);
  const h1 = document.querySelector('h1');
  if (h1) push('h1 copy', (h1.innerText || h1.textContent || '').trim().slice(0, 80));
  return hits;
})()
```

Read the `h1` yourself against the pattern West and Yusuf both name: headline copy that would fit ten thousand other products ("Build faster. Ship smarter."). Copy naming nothing specific is the strongest sameness tell there is, and no probe can score it.

### 1.8 Loading and stability

Core Web Vitals thresholds are Google's published "good" boundaries, assessed at the **75th percentile** of real loads split by device (web.dev). A single automated load is not a Core Web Vitals measurement — it is one sample from a fast machine. Report lab numbers as lab numbers.

| Metric | Good | Poor |
|---|---|---|
| LCP | ≤ 2.5 s | > 4.0 s |
| INP | ≤ 200 ms | > 500 ms |
| CLS | ≤ 0.1 | > 0.25 |

```js
new Promise(res => {
  const out = { lcpMs: null, lcpEl: null, cls: 0, shifts: [] };
  new PerformanceObserver(l => { const e = l.getEntries().at(-1);
    out.lcpMs = Math.round(e.startTime);
    out.lcpEl = e.element ? e.element.tagName + '.' + String(e.element.className || '').slice(0, 30) : null;
  }).observe({ type: 'largest-contentful-paint', buffered: true });
  new PerformanceObserver(l => l.getEntries().forEach(e => {
    if (e.hadRecentInput) return;
    out.cls += e.value;
    out.shifts.push({ v: +e.value.toFixed(4), t: Math.round(e.startTime),
      src: (e.sources || []).map(s => s.node && s.node.tagName).filter(Boolean).slice(0, 3) });
  })).observe({ type: 'layout-shift', buffered: true });
  setTimeout(() => { out.cls = +out.cls.toFixed(4); res(out); }, 4000);
})
```

INP needs real interaction. If the host can click, click the primary CTA and the nav, then read `performance.getEntriesByType('event')` for `duration`. Otherwise report INP as not measured rather than estimating it.

---

## Pass 2 — Rank

Now interpret. Produce **one strictly ordered list** — not buckets. Severity buckets let everything pile into "major" and tell the reader nothing. Force a total order: if two findings feel equal, decide which you would fix first and that one goes higher.

Order by: *how much does fixing this change the experience of the next thousand visitors, per hour of work?* So a low-contrast body font across every page outranks one undersized icon button, and an unclear headline usually outranks both.

Each finding takes exactly this shape:

```
#3  [Defect · WCAG 2.2 SC 1.4.3 AA]  Body copy fails contrast site-wide
    Evidence: .text-slate-400 (#94a3b8) on #ffffff = 2.56:1 at 16px (needs 4.5:1); 47 nodes.
    Fix:      slate-400 → slate-600 (#475569) = 7.58:1.
    Cost:     one token.

#5  [Judgment]  Hero headline does not name the product
    Evidence: h1 reads "Build faster. Ship smarter." Pass 0 impression was
              "some kind of developer tool", still unresolved after a full read.
    Fix:      Name what it does in the h1. Needs a human decision on positioning;
              I can draft options, not choose.
    Cost:     a conversation.
```

Rules for the list:

- **Defect** requires a named success criterion or published threshold (SC 1.4.3, SC 2.5.8, Core Web Vitals). No citation, no Defect.
- **Judgment** must state reasoning, not preference. "Too cramped" is not a finding; "list items sit at 4px gap against 24px section padding, so grouping reads as accidental" is.
- Evidence quotes probe output. If you did not measure it, you cannot cite it.
- Every finding names a fix and a rough cost. A finding without a fix is a complaint.
- Cap at **10**. Extras go to a flat "also noted" appendix with no analysis. A 40-item review does not get read.

### The headline

The review opens with **one sentence** naming the single highest-leverage change — before the list, before any preamble:

> **The one thing:** body text is `#94a3b8` on white throughout, failing AA at 2.56:1 — one token lifts legibility on every page.

If you cannot write that sentence, you have not finished Pass 2.

### Thresholds that are mine, not a standard's

Say so whenever you use these. Defensible starting points, not facts.

- **Body text ≥ 16px.** WCAG sets no minimum font size. Butterick puts optimal web body at **15–25px**; 16px is the low end and the common browser default. [judgment, anchored on Butterick]
- **Measure 45–90 characters** (Butterick); Bringhurst's older convention is 45–75, 66 ideal for single-column. Past ~100 the return sweep starts costing the reader. [cited range; judgment beyond it]
- **Line height 1.4–1.6 for body.** Butterick: 120–145% of point size; SC 1.4.12 requires the page survive 1.5. Tight leading hurts more on long measure. [judgment within cited range]
- **Colour count has no threshold.** Do not assert one. Read the distribution the 1.6 probe returns: a token system shows clustering, a handful of values each used on many nodes. No system shows a long tail of near-duplicates, four greys a step apart, three blues that were each picked once. Report the shape, not a count. [judgment]
- **Spacing off a 4 or 8px base.** Convention, not standard. [judgment]
- **44px touch targets** is **AAA** (SC 2.5.5). AA is **24px** (SC 2.5.8). Always say which bar you are holding the page to.

---

## Pass 3 — Repair

Only if the caller asked for fixes and repo access exists. Reviewing is complete work on its own; do not start editing to seem thorough.

**Before the first edit:** reproduce the finding in source. Grep the measured value (`#94a3b8`, `text-slate-400`, `gap-1`) and read the file. If a finding does not map to a line you can point at, do not fix it — report it and hand back.

- **One finding per edit.** Two fixes in one diff means neither can be reverted on its own, and a reviewer has to untangle which change did what.
- **Fix the token, not the instance.** If 47 nodes share a failing colour, change the variable. If there is no variable, "this needs a token" is itself a legitimate finding.
- **Never touch what you did not measure.** No tidying, reformatting, renaming, dependency bumps, no drive-by corrections to code that happens to sit nearby. Every out-of-scope change makes the diff harder to trust and buries the real fix.
- **Re-verify in the browser after each edit**, re-running the probe that produced the finding. A contrast fix that was not re-measured is not a fix.
- **Check the neighbours.** Darkening a token can fail contrast the other way on a dark surface. Re-run 1.2 whole, not just the failing node.
- If the correct fix is a refactor, propose it rather than perform it.

Leave the page's intent alone. You are correcting execution, not redesigning. Changing a colour that fails contrast is repair; changing a brand colour that passes is trespass.

---

## Stop and hand back

Stop, report what you have, and ask — do not proceed on your own judgment:

- **Anything behind auth, payment, or a destructive action.** Do not create accounts, submit forms with real-looking data, or click anything that deletes, sends, purchases or publishes.
- **Brand-level decisions** — palette, typeface, name, voice, positioning. You can show the current choice reads as generic; you cannot pick the replacement.
- **Copy rewrites** beyond obvious typos. Draft options; let a human choose.
- **A finding you cannot locate in source.** Report the symptom.
- **More than ~10 real defects.** That is not a review finding, it is a signal the page needs a design pass rather than a patch. Say so plainly.
- **The page did not load, never settled, or sits behind a bot wall.** Report the failure; do not review a spinner.
- **Any probe you could not run.** Name it as not-measured. Never present an unmeasured value as measured.

## Reporting

**The one thing** → Pass 0 impression verbatim → ranked findings (≤10) → also-noted appendix → what was not measured and why → screenshots at 390 and 1440, plus 320 if reflow failed.

Say what you are confident about and what you are not. A review honest about its three uncertain calls is worth more than one that sounds certain about all ten.

---

## License

MIT.

Written from primary sources: W3C WCAG 2.2 Understanding documents, Google's
published Core Web Vitals thresholds, Matthew Butterick's *Practical Typography*,
and public commentary on generated-design sameness (Adam Wathan on Tailwind's
`indigo-500`, Alan West, prg.sh, 925Studios). Those are cited inline where their
numbers are used. No third-party skill was adapted.
