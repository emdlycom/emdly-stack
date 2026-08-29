---
name: skill-author-guide
owner: promptsmith
category: Skills
description: How to write an emdly skill that passes review and actually works in an agent — structure, rules that bind, output contracts, and the mistakes reviewers bounce.
version: v3
license: MIT
updated: 2026-08-29
recommended: true
security_checked: true
url: https://emdly.com/skills/promptsmith/skill-author-guide
raw: https://emdly.com/raw/promptsmith/skill-author-guide.md
install: npx @emdly/cli add promptsmith/skill-author-guide
---

# Skill author guide

A skill is a contract between you and an agent. This one explains how to write a contract the agent keeps — and that passes emdly review the first time.

## When to use
- Before writing your first skill. Load it into your agent and ask it to draft one with you.
- When a submission was bounced — the review checklist at the end is what reviewers use.

## Anatomy of a skill that works
1. **Title** — the H1 is the skill's name in words.
2. **One-paragraph purpose** — what job it does and what it refuses to do. If this paragraph could describe five different tools, keep writing.
3. **When to use** — 2–4 concrete situations. Agents use this to decide whether to load you at all.
4. **Input** — what the skill expects to be given. Naming the inputs is half the reliability.
5. **Process** — numbered steps in the order the agent should take them. Each step produces something the next step uses.
6. **Rules** — the binding constraints. See below.
7. **Output format** — a literal example in a code block. Agents copy shapes far better than they follow descriptions.
8. **Edge cases** — the three inputs that would tempt a shortcut, and what to do instead.
9. **License** — a `## License` section with an SPDX id. MIT is the default.

## Rules vs. preferences
A rule is something the agent must do even when it seems unhelpful in the moment. Write rules as absolutes with the reason: "Never invent acceptance criteria — propose them and label the proposal." A preference ("prefer short sentences") is fine but goes in a separate line; mixing the two teaches the agent that rules are negotiable.

The rules that matter most in practice:
- **What not to invent.** Name the things that must come from the input (numbers, quotes, file names, decisions).
- **What to do with uncertainty.** "Flag it" beats "guess" every time; say how to flag.
- **Scope of action.** If the skill can run commands, say which ones and which need a human.

## Output contracts
Give a real example, not a schema. Include the hard parts: an empty case, a flagged item, a number with its source. Then test it: run the skill on three inputs and check that the output matches the example's *shape* exactly. If it drifts, the example is ambiguous — fix the example, not the prose.

## What reviewers bounce
- No license line.
- A prompt, not a skill: one paragraph of "you are a helpful…" with no process or output.
- Instructions that override the visible rules, hidden text, or anything that sends data outside the task. These are blocked automatically.
- Urgency or persuasion tricks in outreach skills ("say the offer ends today").
- Credentials in the file. Skills are public; keys live in the agent's configuration.
- Copy-pasted tool output instead of instructions.

## Review checklist (self-check before submitting)
- [ ] Someone who has never seen my tooling could run this from the file alone.
- [ ] Every "never" has a reason and an alternative.
- [ ] The output example includes a flagged/empty case.
- [ ] Nothing in the file asks the agent to fetch, post or contact anything the task did not mention.
- [ ] `## License` present.

## License
MIT
