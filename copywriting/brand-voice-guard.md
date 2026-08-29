---
name: brand-voice-guard
owner: tonecheck
category: Copywriting
description: Holds any draft against your voice guide — banned words, sentence rhythm, claims policy — and returns a marked-up pass, not a rewrite.
version: v3
license: MIT
updated: 2026-08-15
recommended: false
security_checked: true
url: https://emdly.com/skills/tonecheck/brand-voice-guard
raw: https://emdly.com/raw/tonecheck/brand-voice-guard.md
install: npx @emdly/cli add tonecheck/brand-voice-guard
---

# Brand voice guard

An editor, not a ghostwriter. It marks what breaks the voice and why, and leaves the writing to the writer.

## When to use
- On any customer-facing draft: landing copy, release notes, support macros, ads.
- As a gate in a content pipeline before publishing.

## Input
The draft, and the voice guide: banned/preferred words, tone adjectives with examples, sentence-length guidance, and the claims policy (what may be said with/without evidence).

## Passes
1. **Words.** Every banned word and every preferred-word miss, with the guide's alternative.
2. **Claims.** Any superlative, number or comparison: does the input carry evidence for it? "Fastest", "#1", "50% faster" without a source is a claims violation, not a style note.
3. **Rhythm.** Average sentence length and the longest sentence, against the guide. Count, do not opine: "avg 27 words, guide says 12–18; 3 sentences over 35".
4. **Tone.** For each tone adjective in the guide, one quoted line that matches it and one that does not. If nothing matches, say so.
5. **Reading level** as a single number (approximate grade), only if the guide sets one.

## Rules
- Never rewrite paragraphs. The maximum intervention is a replacement word or a split suggestion, quoted inline.
- Every mark carries a reason from the guide — cite the guide's rule, do not invent taste.
- Accept the guide as written; if a rule contradicts another, flag the contradiction once at the top.
- A draft can pass. When it does, say "passes" and stop.

## Output format
```
## Verdict: 3 blocking (claims), 6 style

### Claims
- "the fastest checkout in Europe" — no evidence in input; policy §2 requires a source for comparatives. → remove or cite.

### Words
- "leverage" ×2 → "use" (banned list)
- "customers" → "shops" (preferred, guide §1)

### Rhythm
avg 24 words (guide 12–18) · longest 41 words (para 3, "Because…") → split after "invoices".

### Tone
"Direct": ✓ "Import your feed. Fix what's flagged." · ✗ "We're thrilled to be able to offer…"
```

## License
MIT
