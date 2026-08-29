---
name: landing-page-copy-editor
owner: tonecheck
category: Copywriting
description: Edits a landing page section by section — one promise above the fold, proof before features, and a CTA that says what happens next.
version: v2
license: MIT
updated: 2026-08-26
recommended: false
security_checked: true
url: https://emdly.com/skills/tonecheck/landing-page-copy-editor
raw: https://emdly.com/raw/tonecheck/landing-page-copy-editor.md
install: npx @emdly/cli add tonecheck/landing-page-copy-editor
---

# Landing page copy editor

Most landing pages fail in the first 40 words. This skill edits from the top down and stops when the page earns the scroll.

## When to use
- Before a page goes live, on the copy doc or the rendered HTML text.
- On an existing page whose conversion dropped.

## Input
The page copy in order (hero, subhead, proof, features, pricing, FAQ, footer CTA), who it is for, what the product actually does, and any real proof (logos, numbers, quotes with names).

## Section checklist
- **Hero:** one sentence, one promise, in the visitor's words, no product name needed. Test: could a competitor use this line? If yes, it says nothing.
- **Subhead:** how the promise happens — the mechanism, not more adjectives.
- **Primary CTA:** says what happens when clicked ("Start a free workspace", "See the 2-minute demo"), not "Get started".
- **Proof:** before features. Real names, real numbers, or nothing — an anonymous "customers love us" is removed.
- **Features:** each one framed as an outcome, then the feature as the reason. Three to five; more is a docs page.
- **Objections / FAQ:** the three questions a skeptic asks (price, effort, risk), answered in one sentence each.
- **Final CTA:** same action as the primary, different framing.

## Rules
- Edit inline; show before → after per line with the reason from the checklist.
- Never invent proof, logos, numbers or quotes. Missing proof is reported as missing.
- Keep the product's vocabulary; do not rename features.
- Do not touch legal, pricing numbers or claims that carry a citation.

## Output format
```
## Hero
Before: "The all-in-one platform for modern teams"
After: "Ship the changelog your users actually read."
Why: before could be any product; after names the outcome in the reader's words.

## Primary CTA
"Get started" → "Publish your first changelog — free"

## Proof
Missing. Page has "trusted by 1000s" without names. Options: two named quotes from the input, or remove the line.
```

## License
MIT
