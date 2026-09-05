---
name: code-documentation
owner: launifycorp
category: Docs &amp; changelogs
description: Documents code so it is still true in a year. Audits what already exists and what in it is false, establishes behaviour by reading the implementation rather than the names, pushes every fact to the closest place it can live — a rename, a type, a validation message, a test — and writes prose only for what the code cannot say.
version: v1
license: MIT
updated: 2026-09-05
recommended: false
security_checked: true
url: https://emdly.com/skills/launifycorp/code-documentation
raw: https://emdly.com/raw/launifycorp/code-documentation.md
install: npx @emdly/cli add launifycorp/code-documentation
---

# Code documentation

Documents code so that the documentation is still true in a year. That constraint decides everything else, because most documentation is not wrong when it is written — it becomes wrong, and the way it was written is what decided how fast.

Two things follow. **Documentation that restates the code is worse than none**, because it doubles the maintenance and starts lying the moment the code changes; `// increment the counter` above `counter++` was never worth its line. And **the only part that lasts is the part the code cannot say**: why it is like this, what was tried and rejected, which of the plausible readings is the one that happens.

So this works in the opposite order to how documentation is usually written. Find out what already exists and what is wrong in it. Establish what the code actually does, by reading it rather than its names. Push every fact as close to the code as it will go. Write prose only for what is left.

## When to use

- A module, package or service nobody outside its author can use.
- Onboarding: the same three questions get asked in chat every time somebody new arrives.
- Before a handover, when the person who knows why is leaving.
- After a refactor, when the docs describe the previous shape.
- Auditing existing documentation to find what is now false.

Not for: an HTTP endpoint reference page (`shiplog/api-reference-writer`), a release changelog (`shiplog/changelog-composer`), explaining one query (`querydeck/sql-query-explainer`), or a commit message (`kernelpanic/commit-message-editor`).

## Input

Required:

- **The code.** A repository, a package, a directory, or a set of files. Reading it is not optional; see Stop 1.
- **Who this is for, and what they are trying to do.** A contributor changing it, a consumer calling it, an operator running it at 3am, or someone deciding whether to adopt it. These four want four different documents and the same words serve none of them.

Useful when offered:

| also useful | what it changes |
|---|---|
| the questions people actually ask | the fastest route to what is missing |
| the last three incidents or bug reports | traps that have already caught someone are proven, not guessed |
| the project's existing conventions | matching them beats importing a better style |
| whether this is public or internal | see Stop 4 |

## The distance rule

**A fact rots in proportion to its distance from the code it describes.** So a fact goes at the lowest rung of this ladder that can carry it, and prose is what is left after the ladder is exhausted.

| rung | rots | who reads it |
|---|---|---|
| 1 · a name | never | everyone, always |
| 2 · a type or signature | cannot, the compiler enforces it | everyone who calls it |
| 3 · a validation error or assertion message | at the moment it is wrong | the person who got it wrong |
| 4 · a test | loudly, in CI | contributors |
| 5 · a docstring | quietly, but travels with the file | callers, in the editor |
| 6 · a module or file header | quietly | someone opening the file |
| 7 · a README beside the code | slowly | someone entering the directory |
| 8 · a document in `docs/` | faster | someone who went looking |
| 9 · a wiki, a ticket, a chat message | fastest, and invisibly | nobody, eventually |

Rename the badly named thing rather than explaining it. Add the type rather than describing the type in prose. Put the constraint in the validation message rather than in a paragraph nobody reads before the call. Write the test rather than the sentence claiming the behaviour.

**Each of those is a fix that also documents. The paragraph is only a document.**

Every placement decision goes in the output, so the reader can disagree with where a fact went rather than only with what it says.

## Which document

For what survives the ladder, use the Diátaxis compass — Daniele Procida's framework, which sorts documentation by two questions: *action or cognition*, and *acquisition or application*.

| the content | serves | so it is |
|---|---|---|
| informs action | acquisition of skill | a **tutorial** |
| informs action | application of skill | a **how-to guide** |
| informs cognition | application of skill | **reference** |
| informs cognition | acquisition of skill | **explanation** |

The practical value is the failure it names: most bad documentation is one type wearing another's clothes. A tutorial that stops to explain the type system loses the beginner. A reference page with a narrative loses the person who came for one parameter. Say which of the four you are writing, at the top, and do not blend them.

**Decisions get their own form.** Why the queue is at-least-once, why the library was written rather than bought, why the obvious approach was rejected — that is `explanation`, and it belongs in a dated, numbered decision record that is never edited, only superseded. A decision rewritten in place loses the thing that made it worth keeping.

## What to document first

Coverage is the wrong target. A codebase with a docstring on every getter and nothing on the ordering requirement between two calls is fully covered and useless.

| priority | what it is |
|---|---|
| `BLOCKS` | without this nobody can use it at all: setup, required configuration, credentials, the first working call |
| `TRAPS` | they will use it and be wrong without knowing: surprising defaults, side effects, ordering requirements, silent truncation, units, time zones, what happens on failure |
| `WHY` | decisions a future maintainer will undo because the reason is invisible |
| `NICE` | everything else, and most of it should not be written |

Work down. A `TRAPS` item on a function used everywhere outranks a `BLOCKS` item on a corner nobody enters — say so when you reorder, and why.

> Thresholds above are defaults; report the thresholds you used.

## Pass 0 — What exists, and what in it is false

Never write next to a wrong document without dealing with the wrong one. Two documents that disagree are worse than one that is missing, because now the reader has to decide, and they will pick the wrong one half the time.

Find: README files at every level, `docs/`, docstrings and comments, type hints, decision records, the wiki if there is one, and the comments in the code that contradict the code.

For each, one verdict: **current**, **stale** (was true, is not now), **wrong** (never was true), or **unverifiable**. Quote the line and the code that disproves it.

A stale document is a finding in its own right, and often the most valuable thing this pass produces.

## Pass 1 — Establish what the code actually does

**Read the implementation.** Not the name, not the docstring, not the test names. Those are three claims about the code, and any of them can be wrong; the code is what runs.

For each thing being documented, establish and note where you found it:

- What it returns, including on every path, and what it returns when given nothing.
- What it raises or errors, and what the caller sees.
- Every side effect: writes, network calls, mutation of arguments, global state, cache, logging.
- Defaults, and where they come from — a signature, an environment variable, a config file, a fallback three layers down.
- Units, time zones, encodings, precision, and whether a number is inclusive or exclusive.
- Ordering and lifecycle requirements: what must be called first, what must not be called twice, what must be closed.
- Concurrency: is it safe to call from two places at once, and how would you know.

**Anything you could not establish is written down as unestablished**, not omitted and not guessed. See Stop 1.

## Pass 2 — Place every fact

For each fact from Pass 1, take the lowest rung on the ladder that can carry it, and say which rung you chose.

Facts that become code changes get proposed as code changes, not as prose:

```
FACT   timeout is in seconds, not milliseconds
RUNG   2 — the parameter is `timeout: int`
FIX    rename to `timeout_seconds: int`, or accept `timedelta`
DOC    then nothing; the signature carries it
```

```
FACT   calling `close()` twice raises rather than being a no-op
RUNG   4 — no test covers it
FIX    add the test; it is the documentation that fails when it changes
DOC    one line in the docstring, because callers read that in the editor
```

## Pass 3 — Write what the ladder could not carry

Now, and only now, prose. It is mostly `WHY` and mostly short.

- **Lead with what the reader came for.** Not the history, not the philosophy.
- **One claim per sentence**, so a wrong one can be fixed without rewriting a paragraph.
- **Name real things**: real function names, real file paths, real values. `foo` and `some_value` are how examples stop matching reality.
- **Say what it does not do.** The boundary is often more useful than the capability, and it is almost never written down.
- **Do not write what the code says.** If your sentence is the signature in English, delete it.
- **Date and attribute anything that is a judgement.** "We chose X because Y, in March 2026, with what we knew then" ages honestly. "X is better than Y" does not.

## Pass 4 — Run the examples

**Every example is executed before it ships.** An example that does not run is a bug report your users file for you, and it is the fastest way to lose their trust in the rest of the document.

- Run it against the current version. State the version it was run against.
- No pseudo-code presented as code. If it cannot be run, mark it as a sketch.
- No credentials, no real tokens, no real customer data, no internal hostnames. See Stop 3.
- If the example needs setup, the setup is part of the example.
- Where you cannot run it — no environment, needs a live service — say so beside the example rather than letting it look verified.

## Rules

- **Verified or marked unverified.** Every behavioural claim traces to a line you read or a run you did. See Stop 1.
- **The code is the authority.** If the code and the existing doc disagree, the doc is wrong until someone says the code is the bug — then it is a bug report, not a documentation task. See Stop 2.
- **Delete more than you add.** A wrong line removed is worth more than a right line added, and this is the pass people skip.
- **Never invent a rationale.** If nobody knows why, write "the reason is not recorded" and name who might know. A plausible invented reason is the most damaging thing in this whole skill, because it will be quoted back as fact for years.
- **Match the project's conventions**, including ones you would not have chosen. Consistency is worth more than your preference.
- **Write for the reader you were given.** A contributor doc and a consumer doc are different documents; do not merge them to save effort.
- **Say what you did not cover** and why, at the end of every output.

## Output format

```
DOCUMENTATION · payments/refunds.py · 412 lines · for consumers of the module
Read at commit 8f2a41c. Examples run against 2.4.1.

WHAT EXISTS
  docstring on refund()        STALE — says "returns None on failure"; it has
                               raised RefundRejected since 2.1 (refunds.py:88)
  README in payments/          CURRENT, but describes the old two-step flow as
                               "recommended"; it was deprecated in 2.3
  docs/payments.md             WRONG — documents a `partial=` parameter that
                               does not exist and never did in this repo
  type hints                   present on 6 of 11 public functions
  decision records             none found

  The stale docstring is the finding here. Anyone reading it writes a `None`
  check that never fires and no error handling that does.

GROUND TRUTH — established by reading, with locations
  refund() returns Refund on success                          refunds.py:71
  raises RefundRejected on a declined refund                   refunds.py:88
  raises nothing on a network timeout — the underlying client
    retries 3 times then re-raises httpx.TimeoutException      client.py:34
  amount is in minor units (cents), never validated            refunds.py:52
  idempotency_key is optional and defaults to None, which
    means the provider will accept a duplicate refund          refunds.py:57
  writes an audit row before calling the provider, and does
    not roll it back if the provider fails                     refunds.py:63
  NOT ESTABLISHED: whether concurrent refunds on one payment
    are safe. No lock in this module, no test, and the
    provider's behaviour is not documented. Named as open.

PLACEMENT
  amount in minor units       rung 2 · rename `amount` → `amount_minor`,
                              or take a Money type. Proposed as a code change.
  no validation on amount     rung 3 · raise on a negative or non-integer
                              amount with a message that says minor units
  double refund without key   rung 5 · docstring, because callers decide this
  audit row not rolled back   rung 5 + WHY · docstring plus a decision record;
                              this looks like a bug and is deliberate
  timeout behaviour           rung 4 · no test covers the retry; add it

PRIORITY
  TRAPS  amount units · idempotency default · audit row on failure
  BLOCKS nothing — the module is importable and the first call works
  WHY    the audit-before-call ordering
  NICE   the five missing type hints

WRITTEN
  refunds.py    docstring on refund(), 9 lines, replacing the stale one
  refunds.py    module header, 4 lines, stating minor units once
  docs/adr/     0007-audit-before-provider-call.md, dated, superseding nothing
  payments/     README section on the deprecated two-step flow, 6 lines

DELETED
  docs/payments.md — the `partial=` section, 14 lines documenting a parameter
  that does not exist. Proposed, not removed; see Stop 5.

EXAMPLES
  2 written, both run against 2.4.1, output pasted verbatim.
  The concurrent-refund case has no example because the behaviour is not
  established.

NOT COVERED
  The provider client in client.py. Different audience — that is a contributor
  document and this one is for consumers. Say the word and it is a separate pass.
  Whether concurrent refunds are safe. Needs someone who can test against the
  provider sandbox.
```

The refusal:

```
NOT DOCUMENTED

I can see the file list and the function names. I have not read the bodies.

Documentation written from names is the specific failure this skill exists to
avoid: it is fluent, it is plausible, and it is wrong in the places that matter,
because a function called `validate_order` may validate nothing and a
`get_user` may write to the database. Someone will trust it more than they
trust the code.

Give me the source, or the parts of it you want documented. If the code cannot
be shared, I can write the structure and the questions it needs answered, and
somebody with access fills in the behaviour — but that document must say it was
written without reading the code.
```

## Edge cases

- **No code, only names or a directory listing.** See the refusal. Do not proceed.
- **Generated code**, migrations, vendored dependencies, lock files. Do not document them individually. One line saying what generates them and how to regenerate is the whole job.
- **Code you cannot run.** Read-only is still fine for Passes 0 to 3; mark every example as unrun and say why.
- **A doc that is right and the code that is wrong.** That is a bug, not a documentation task. See Stop 2.
- **Two documents that disagree with each other.** Report both, say which the code supports, and do not silently pick one.
- **A comment that contradicts the code.** The highest-value finding available. Quote both, name the line, and say which is likely stale rather than which is "right".
- **Dead code.** Do not document it into permanence. Report it as apparently unreachable and let someone decide; documenting it makes it harder to delete.
- **A codebase too large for one pass.** Do not skim it all. Pick by the priority table — the modules that block or trap people — document those properly, and name what you left.
- **A language whose conventions you know less well.** Match the surrounding code, say you did, and flag anything idiomatic you may have misread.
- **Auto-generated API docs already exist.** They are rung 2 and 5, already done. Your job is what they cannot express: why, ordering, traps, the shape of the whole.
- **The only person who knew why has left.** Write "the reason is not recorded" and name the commit, the PR, or the ticket that might carry it. Never fill the gap.
- **Documentation in a language other than the code's comments.** Ask which the project uses; do not introduce a second one.

## Stop and hand back

1. **Never document behaviour you have not verified.** Not from a function name, not from a test name, not from an existing docstring, not from what the framework usually does. Fluent invented documentation is worse than a gap, because a gap gets filled by reading the code and a wrong line gets trusted. Where you could not establish something, write that it is unestablished and say what would settle it.
2. **Never write documentation that papers over a bug.** If the behaviour is wrong, documenting it makes it permanent and makes the next person's correct fix look like a regression. Report it as a bug, say what you would have written, and let someone decide.
3. **No secrets in examples.** No real tokens, keys, connection strings, customer records or internal hostnames — including in the output you paste. Placeholders that are obviously placeholders.
4. **Internal, unreleased, or security-relevant detail heading somewhere public.** Rate limits and their bypasses, auth internals, admin endpoints, an unannounced feature. Ask where the document will live before writing it, and if the answer is public, hand that decision to whoever owns the release.
5. **Do not delete someone else's documentation.** Propose the deletion with the reason and the evidence. A page that looks abandoned may be the one thing a team relies on, and the author is not in the room.
6. **Licence, compliance, security or safety documentation.** You can describe what the code does. Whether a statement satisfies an obligation belongs to whoever holds that obligation.
7. **A rationale nobody can confirm.** See Rule 4. Asking the person who wrote it is the correct move and it is not your call to skip.

## Sources

- The four documentation types and the compass are **Diátaxis**, by Daniele Procida, `diataxis.fr`. The compass sorts content by *action or cognition* and *acquisition or application*; the table above is its own.
- Everything else here is method rather than standard. The distance ladder, the four priority levels and the rule that examples must be run are this skill's judgement, and a project with a different shape should adapt them.

## License

MIT
