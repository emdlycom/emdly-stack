---
name: skill-author-guide
owner: promptsmith
category: Skills
description: How to write an emdly skill that passes review and actually works in an agent — structure, rules that bind, output contracts, and the mistakes reviewers bounce.
version: v4
license: MIT
updated: 2026-08-30
recommended: true
security_checked: true
url: https://emdly.com/skills/promptsmith/skill-author-guide
raw: https://emdly.com/raw/promptsmith/skill-author-guide.md
install: npx @emdly/cli add promptsmith/skill-author-guide
---

# Skill author guide

A skill is a contract between you and an agent. The agent cannot ask you what you meant, so every clause has to be checkable from the file alone: a section it can find, a number it can compare against, a shape it can copy. This guide is the emdly house standard written out — what the file must contain, in what order, and what a reviewer will diff your submission against. Follow it and your skill passes review the first time.

## When to use
- Before writing your first skill. Load this file into your agent and draft alongside it.
- When revising an older skill to the current standard — most pre-2026 files fail sections 3, 4 and 6 below.
- When a submission was bounced. The review checklist at the end is what reviewers run.
- When you are reviewing someone else's skill and need a shared yardstick.

## Input and output format for this file
This guide has no `## Input` and no `## Output format` section, and that is deliberate. Those sections describe the data a task skill consumes and the artifact it emits. This file is a guide: it is read by a human or loaded as context by an agent that is about to write a different file. Its output is that other file, and the whole of "The worked example" below is the demonstration. Every *task* skill you write must have both sections. If you are writing a guide, a glossary or a reference and you drop them, say so in one sentence the way this paragraph does. Silent omission reads as an oversight; a stated omission reads as a decision.

## Section order
Use these headings, in this order, with these names. Reviewers look for them literally.

```
# Skill name                sentence case; the word "skill" does not appear in it
<thesis paragraph>          one paragraph: who the output is for, and what must be
                            true of it. Not a feature list.
## When to use              3-5 bullets, concrete triggers
## Input                    exactly what must be supplied, and in what shape
## <process>                rules or a named process. "Five passes", "The ritual",
                            "Section checklist" beat a generic "Process".
## Output format            the shape, then a fully filled example
## Edge cases               mandatory. The inputs that tempt a shortcut.
## Stop and hand back       required where the trigger list below applies
## License                  an SPDX id. MIT is the default.
```

No YAML frontmatter. Emdly reads the H1 and the first paragraph; a frontmatter block is dead weight the agent still pays tokens for.

## Rules that bind
A rule an agent cannot check is decoration. Before you keep a rule, ask what the agent compares against to know it obeyed. Every rule carries one of four things: a number, a named format, a closed list, or an explicit refusal condition.

| Replace | With |
|---|---|
| "be concise" | "under 150 words, excluding numbered steps" |
| "sound human" | a banned-word list, or a named cadence |
| "use good judgment" | the condition, and what to do on each side of it |
| "keep it to one screen" | "under 40 lines" |
| "a real angle" | a closed list of what counts as one |

Bound the exceptions too. "Under 150 words unless the answer needs steps" hands the agent its own escape hatch. "150 words excluding numbered steps, steps ≤ 8" does not.

Keep rules and preferences apart. A rule is what the agent must do even when it looks unhelpful in the moment, and it is written as an absolute with its reason: "Never invent acceptance criteria — propose them and label the proposal." A preference is fine, but put it on its own line under a preferences heading. Mixing the two teaches the agent that rules are negotiable.

The three rules almost every skill needs:
- **What not to invent.** Name what must come from the input: numbers, quotes, file names, dates, decisions.
- **What to do with uncertainty.** "Flag it" beats "guess", and you must say how to flag — the literal string the agent writes.
- **Scope of action.** If the skill can run commands or send anything, name which commands, and which need a human first.

## Every number is sourced, disclaimed, or refused
Three legal forms. There is no fourth, and a bare assertion is the most common reason a skill gets bounced.

1. **Cited.** Name the standard, criterion, spec or vendor doc. "Horizontal scroll at 320px fails WCAG 2.2 SC 1.4.10." "Merchant Center caps `description` at 5000 characters."
2. **Disclaimed.** Tag it `[judgment]` and say what it is anchored on. "Body text ≥ 16px — WCAG sets no minimum; Butterick puts optimal web body at 15-25px, 16 is the low end and the common browser default. [judgment, anchored on Butterick]"
3. **Refused.** If no defensible number exists, say so and describe the shape to read instead. "Colour count has no threshold. Do not assert one. Read the distribution."

If a number is a house convention rather than a finding, write "house rule" next to it — that is honest and it survives review. If the skill runs on tunable thresholds, add this line verbatim:

> Thresholds above are defaults; report the thresholds you used.

## The worked example carries the hard cases
The example is what agents copy. They copy shapes far more reliably than they follow prose, which means an example that shows only the happy path teaches the skill to produce only happy paths.

- No elision. No `…`, no `(3 items)`, no `Body: …`. If the deliverable is email copy, the example contains email copy. If it is a table, every cell is filled.
- **Show the empty case.** `no recorded activity` · `—` · `(not stated)` · `too small to judge` · `unusable notes: 9` · `not found`.
- **Show the flagged case.** If a rule says to mark something "undocumented — ask the owner", a line in the example says exactly that string.
- **Show the refusal.** If a rule says the skill declines on missing input, show what the decline looks like on the page.
- **Check the arithmetic.** Recompute every figure. Totals must equal their parts, percentages must divide, dates must be consistent. Examples whose numbers do not reconcile teach the agent that numbers are decorative.

Use a second example when one shape genuinely does not cover the range — a full input and a thin one, for instance. Do not use a second example to say the same thing twice.

## Failure handling
`## Edge cases` is mandatory. Cover, wherever they apply: input absent · input malformed · input empty · input too large to process · a required secondary source unavailable (no code access, no changelog, no voice guide, no policy).

The rule underneath all five: never let the skill degrade silently into a worse method. If the method collapses without something, say the method collapses, and say what to report instead. A reachability check with no code access is not a slightly weaker reachability check; it is a CVSS sort, which is the thing the skill exists to refuse.

## When you need a "Stop and hand back"
Required when the skill touches money, hiring or any other decision about a person, production systems, customer-facing output that ships without further review, security findings, or anything with a legal or regulatory edge.

Write named triggers, not a general caution. Each trigger says what to do instead and who decides. A stop is not a warning in prose — it halts. Good ones in the catalog: `sevzero/k8s-incident-triage` pairs a read-only allowlist with "the skill itself never executes anything"; `helpdeskly/support-reply-drafter` lists "legal threats, security or data-loss reports, anything about a minor, a customer who has written three times without resolution, or profanity aimed at a person".

If none of the triggers apply, omit the section. Do not add an empty one to look thorough.

## Length
Target 120-200 lines. Expansion must be substance: the step that currently says "read" or "find" without saying where and what counts, the edge cases, the sourcing, a second example where the range needs it. Do not pad with restatement, generic advice, or a closing summary. If a file is genuinely complete at 40 lines, adding the missing sections is the whole job.

## Voice
Direct, second person or imperative. Short sentences. Name things concretely. No hedging, no marketing, no em dashes as a tic. Never "delve", "robust", "comprehensive", "leverage", "seamlessly".

## What reviewers bounce
- A prompt, not a skill: a paragraph of "you are a helpful…" with no process and no output format.
- Instructions that override the visible rules, hidden text, or anything that sends data outside the task. These are blocked automatically, not bounced.
- Urgency or persuasion tricks in outreach skills ("say the offer ends today").
- Credentials in the file.
- Copy-pasted tool output standing in for instructions.
- No `## License`.

## The worked example
A whole short skill, end to end. It is deliberately small, and it still carries a cited number, a disclaimed one, a refused one, an empty case, a flagged case, a refusal, and a stop.

````
# Alt text writer

Alt text is read aloud to someone who cannot see the image, in the middle of a
sentence. This skill writes that sentence for marketing-page images and refuses
to write one for images it cannot see.

## When to use
- Before a marketing page ships, on every `<img>` in the page source.
- When a CMS export lists images with empty or placeholder `alt` values.
- When auditing an existing page against WCAG.

## Input
For each image: the file or URL, the surrounding paragraph, and whether the
image is a link. Alt text cannot be written from a filename alone.

## Rules
- Describe what the image conveys in context, not what it depicts. The same
  photo needs different alt text in a pricing page and a careers page.
- Decorative images get `alt=""`. An image is decorative only when removing it
  loses nothing — WCAG 2.2 SC 1.1.1 treats pure decoration as exempt.
- Alt text ≤ 125 characters. WCAG sets no limit; 125 is where common screen
  readers stop reading without a pause. [judgment, anchored on screen-reader
  behaviour, not on the spec]
- There is no threshold for how many images a page "should" have alt text on.
  Every non-decorative image does. Do not report a percentage.
- Never start with "image of" or "picture of". The screen reader already said it.
- Never invent detail you cannot see. Write `cannot see image — ask the owner`.

Thresholds above are defaults; report the thresholds you used.

## Output format
```
| # | file | alt | note |
| 1 | hero-team.jpg | Four engineers at a standing desk reviewing a deploy dashboard | 62 chars |
| 2 | divider-wave.svg | "" | decorative |
| 3 | chart-q3.png | Q3 signups rose from 1,200 to 4,100 | number read from the caption |
| 4 | logo-acme.png | cannot see image — ask the owner | file 404s in the export |
| 5 | banner.webp | — | image is a link with no destination given: declined, see below |

Declined: image 5 is wrapped in an `<a>` whose href is absent from the input.
Alt text for a linked image describes the destination, not the picture, so
there is nothing defensible to write. Supply the href and re-run.
```

## Edge cases
- **No surrounding paragraph.** Context is the method. Write the alt from the
  image alone, mark the row `no context — verify in place`, and say in the
  summary how many rows are marked.
- **Image unreachable.** Row reads `cannot see image — ask the owner`. Never
  guess from the filename.
- **Page has no images.** Say "no images found" and stop. Do not report success.
- **More than 200 images.** Do the first 200 in page order, then say how many
  remain. A truncated table that admits it beats a rushed full one.
- **Text baked into the image.** The alt must contain that text verbatim. If it
  is longer than 125 characters, say so and recommend real text instead.

## Stop and hand back
Halt and name a human before the page ships when:
- The image carries a price, a legal disclaimer, or a claim about the product.
  The words are the thing being published; marketing owns them, not this skill.
- The image shows an identifiable person. Whether to name them is theirs to
  decide, not yours.
- The image is a chart whose numbers do not appear anywhere in the input.

## License
MIT
````

Every mandated branch appears above: a citation (SC 1.1.1), a `[judgment]` anchor (125 characters), a refusal (no percentage threshold), an empty case (`""` and `—`), a flagged case (`cannot see image — ask the owner`, the literal string the rule named), a decline written out in full, and a stop with three named triggers.

## Edge cases
- **You have no defensible number for a rule.** Do not round one up. Use form 3: refuse it, and describe the shape the agent should read instead.
- **Your skill has no empty case.** It almost certainly does — think about what happens when the input is present but says nothing. If it truly has none, say so in `## Edge cases` in one line.
- **You are revising a skill, not writing one.** Keep the H1, the owner's voice, the category and the license line verbatim. Add the missing sections rather than rewriting what already works.
- **The skill wraps a tool you cannot describe.** Name the tool, its version, and what the agent should do when it is unavailable. A skill that assumes a tool it never names is unrunnable by anyone but you.
- **A rule you believe in resists being made checkable.** That is usually a sign it is two rules. Split it and check each half.

## Review checklist
- [ ] Every section above is present, in order, with those names.
- [ ] Someone who has never seen my tooling could run this from the file alone.
- [ ] Every rule carries a number, a named format, a closed list, or a refusal condition.
- [ ] Every number is cited, `[judgment]`-tagged with its anchor, labelled a house rule, or refused.
- [ ] The example has no elision, and shows the empty, flagged and refused cases.
- [ ] I recomputed every figure in the example and it ties out.
- [ ] `## Edge cases` covers absent, malformed, empty, oversized, and missing secondary source.
- [ ] `## Stop and hand back` is present if any trigger applies, and absent if none do.
- [ ] Every "never" has a reason and an alternative.
- [ ] Nothing in the file asks the agent to fetch, post or contact anything the task did not mention.
- [ ] No credentials. Skills are public; keys live in the agent's configuration.
- [ ] `## License` present with an SPDX id.

## License
MIT
