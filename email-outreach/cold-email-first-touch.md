---
name: cold-email-first-touch
owner: outboundlab
category: Email & outreach
description: Writes first-touch e-mails that sound like a person — one observation, one sentence of value, one soft ask. Kills every buzzword on sight.
version: v3
license: MIT
updated: 2026-08-28
recommended: false
security_checked: true
url: https://emdly.com/skills/outboundlab/cold-email-first-touch
raw: https://emdly.com/raw/outboundlab/cold-email-first-touch.md
install: npx @emdly/cli add outboundlab/cold-email-first-touch
---

# Cold e-mail, first touch

The only cold e-mail that gets answered is the one that could not have been sent to anyone else. This skill enforces that.

## When to use
- Per prospect, with research notes as input.
- For a sequence, this is mail 1 only; follow-ups are a different skill.

## Input
Prospect: name, role, company, and **one verifiable observation** (a job post, a product change, a talk, a public number). Sender: who you are and the one thing you do. A single case in one sentence with a number if you have one.

## Structure — four lines, in this order
1. **Observation.** Something true about them that took effort to know. Quote or cite it.
2. **Bridge.** One sentence connecting the observation to a problem you solve — stated as a question or a hypothesis, not a claim.
3. **Proof.** One sentence: who else, what changed, one number.
4. **Ask.** Small and reversible: "worth a 15-minute look?" / "should I send the two-pager?" Never "book a time on my calendar".

## Rules
- Under 60 words of body. Subject under 5 words, lowercase is fine, no "Quick question".
- No flattery, no "I hope this finds you well", no "I'd love to", no "synergy", "leverage", "streamline", "solution", "innovative", "revolutionary", "game-changer", "circle back".
- No fake familiarity ("saw you're crushing it"). No reply-bait ("did you see my last mail?" on a first touch).
- If the input lacks a verifiable observation, do not write the mail. Return "no observation — research first" and say what to look for.
- One link at most, and only in the proof line.
- Sign with a name and a role. No banners, no calendar links, no PS.

## Output format
```
Subject: your PostgreSQL 13 migration

Hi Mara — your team posted a Postgres 16 migration role last week, so I assume the 13 → 16 jump is on the roadmap.

If the blocker is the downtime window, that is what we do: Lumen cut theirs from 6 h to 40 min last quarter.

Worth a 15-minute look?

Sean Callahan · Outbound Lab
```

## License
MIT
