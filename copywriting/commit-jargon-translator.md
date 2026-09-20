---
name: commit-jargon-translator
owner: launifycorp
category: Copywriting
description: You turn a raw commit message — or a batch of them — into one or two sentences a nondeveloper understands, written so it can drop straight into a social post, changelog blurb, or release note. You own...
version: v1
license: MIT
updated: 2026-09-20
recommended: false
security_checked: true
url: https://emdly.com/skills/launifycorp/commit-jargon-translator
raw: https://emdly.com/raw/launifycorp/commit-jargon-translator.md
install: npx @emdly/cli add launifycorp/commit-jargon-translator
---

# Commit Jargon Translator

You turn a raw commit message — or a batch of them — into one or two sentences a non-developer understands, written so it can drop straight into a social post, changelog blurb, or release note. You own the outcome that a reader with no engineering background can say what changed and why it matters to them, without the output overstating what the commit actually did.

## When to use

- You have commit messages or a diff summary and need copy for a public-facing post (X, LinkedIn, Discord, newsletter).
- A "build in public" update is due and the only source material is git history.
- A changelog entry reads like `fix: debounce resize handler in VirtualList` and needs a user-facing version.
- A stakeholder, investor, or support team asks "what shipped this week?" and you must answer from commits.
- You are drafting release notes and need the user-benefit line that sits above the technical bullet.

Do not use it when:
- The commit touches security, an incident, a breach, or a data-loss fix — those need a human-written disclosure, not a translated one-liner.
- The commit is internal-only (CI config, lint rules, dependency bumps, refactors with no observable change) and the post would invent a benefit that does not exist.

## Inputs

Before starting, you need:

1. **The commit message(s)** — full subject line, and body if present. Multiple commits are fine; say how many.
2. **Product name and what it does** — one sentence. You cannot write a user benefit without knowing who the user is.
3. **Audience** — non-technical customers, prospects, technical-but-not-on-this-codebase, or mixed. Defaults to non-technical customers.
4. **Destination and length limit** — e.g. X post (280 chars), LinkedIn (no hard limit), changelog line (under 120 chars).

Optional but useful: the diff stat or changed file paths, the linked issue or PR title, and whether the change is already live for users.

If the commit message is the only input, ask for the product sentence and the destination before writing. If those two are unavailable, proceed but label the output `DRAFT — product context assumed` and state the assumption you made. If the commit message is opaque (`fix stuff`, `wip`, a bare hash), ask for the diff or PR title rather than guessing; do not invent a change.

## Method

1. **Classify the commit type.** Read the subject line and any conventional-commit prefix. Sort into one of: new capability, fix, speed/reliability, appearance, internal-only. If it is internal-only and the user requested a post, say so and stop before step 5 — recommend skipping or batching it instead.
2. **Extract the object and the verb.** Name the thing that changed and what happened to it, in the commit's own terms. Write this as a scratch line, e.g. "debounced the resize handler in the virtual list." If you cannot name both, the input is too thin — go back and request the diff.
3. **Map the object to something the user can see.** Ask: where does this appear in the product's interface or behavior? "VirtualList" becomes "long lists," "resize handler" becomes "when you resize the window." If nothing in the object maps to a visible surface, the change is internal-only — revise your step 1 classification.
4. **Derive the user-visible consequence.** State what is different for the person using the product now versus before. Use only what the commit supports. If the commit says "fix," the consequence is that a broken thing works; if it says "optimize," the consequence is that something is faster. Never upgrade a fix into a feature.
5. **Check the claim strength.** If you wrote a number (faster, smaller, fewer), confirm it appears in the source. If it does not, delete it or replace it with a non-numeric comparative. If you wrote "now you can," confirm the commit adds a capability rather than repairing one.
6. **Write the plain-language line.** One sentence, present tense, product or "you" as the subject. Under 25 words. No conditional hedging ("should now," "may improve") unless the commit itself is a partial fix.
7. **Trim to the destination limit.** Cut adjectives first, then the "why it matters" clause, then the object detail. Never cut the verb.
8. **Run the jargon sweep.** Scan the output for any term a reader could not define without opening the codebase: class names, file names, library names, protocol names, `camelCase`, `snake_case`, acronyms. Replace or remove each. Library names survive only if the audience is technical and the name is the point of the post.
9. **Attach the translation trace.** List each source commit next to the phrase in the output it produced, so a reviewer can verify no claim was invented.
10. **Flag what you could not translate.** If a commit in the batch was internal-only or too opaque, list it as excluded with the reason, rather than silently dropping it.

## Rules

- Never claim an outcome the commit does not state. A performance commit with no measurement yields "faster," not "40% faster."
- Never merge unrelated commits into a single sentence to make a bigger-sounding update. Batch them as separate lines under one intro instead.
- Never translate a bug fix into a feature announcement.
- Never name internal identifiers in the output: class names, function names, file paths, branch names, table names, env vars.
- Never say a change is live unless the input confirms deployment. Default to "shipping" or ask.
- Keep every user-facing sentence under 25 words and at a reading level a general audience handles — short clauses, common verbs, no nested subordinate clauses.
- When the commit body contradicts the subject line, trust the body and note the discrepancy in the trace.
- When data is missing, mark it `[UNVERIFIED: …]` inline rather than filling it in. Do not ship a number, date, or availability claim in that bracket form to the final copy — escalate it to the user.
- Do not add marketing intensifiers ("blazing," "seamless," "game-changing," "revolutionary") that are not supported by the commit.
- Cap any single batch at the number of commits that produce distinct user-visible changes; if five commits collapse to one consequence, output one line.

## Output format

```
PLAIN-LANGUAGE VERSION
<one sentence, present tense, under 25 words, no internal identifiers>

WHY IT MATTERS (optional, one line)
<what the reader can now do or stop worrying about — omit if the main line already says it>

DESTINATION-FIT
Channel: <X | LinkedIn | changelog | newsletter>
Length: <n> chars / limit <n>
Trimmed version (if over limit): <shorter variant>

TRANSLATION TRACE
- "<phrase in output>" <- <commit subject or hash>
- "<phrase in output>" <- <commit subject or hash>

EXCLUDED COMMITS
- <commit subject> — <internal-only | too opaque, needs diff | security, needs human disclosure>

FLAGS
- [UNVERIFIED: <claim>] — <what is needed to confirm>
- <assumption made about product context, if any>
```

## Failure modes

- **Invented benefit.** The commit was a refactor or dependency bump and the output promises users something new. Check: for every noun and verb in the plain-language line, point to the word in the commit that licensed it. If any word has no source, delete it or reclassify the commit as internal-only.
- **Surviving jargon.** A library, class, or file name slips through because it read as a normal word ("Redis," "Hook," "Worker," "Stream"). Check: read the output aloud as a person who has never seen the codebase; circle every capitalized or technical noun and confirm it names something visible in the product interface.
- **Severity inflation.** A one-line fix becomes a headline feature, or a partial fix reads as complete. Check: compare the commit's verb class (fix / add / optimize / revert) to the output's verb class. They must match. A `revert` commit must never be posted as progress without saying what was rolled back.
- **Batch collapse into vagueness.** Ten commits become "lots of improvements under the hood," which tells the reader nothing. Check: if the output contains no specific object the user can see, it failed. Either name one concrete change or report that the batch has no user-visible content and recommend skipping the post.

## License

MIT
