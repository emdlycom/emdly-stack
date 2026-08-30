---
name: geo-audit
owner: launifycorp
category: SEO
description: Generative engine optimisation audit from one URL: which AI crawlers are permitted and what each operator says that means, what a non-rendering fetch actually receives, whether any passage is self-contained enough to be lifted, and whether anything on the page could only have come from there. Reports configuration, never a visibility score, because none exists.
version: v1
license: MIT
updated: 2026-08-30
recommended: false
security_checked: false
url: https://emdly.com/skills/launifycorp/geo-audit
raw: https://emdly.com/raw/launifycorp/geo-audit.md
install: npx @emdly/cli add launifycorp/geo-audit
---

# GEO audit

Give it a URL. It reports whether generative engines can reach the page, whether they can lift a clean answer out of it, and whether they have any reason to name the source when they do.

Generative engine optimisation is not search optimisation with new vocabulary. The unit that ranks in search is a page; the unit that gets used in an AI answer is a **claim** — one self-contained, attributable statement that a model can lift, compress and footnote. A page can rank well and contribute nothing, because everything worth quoting on it is spread across four paragraphs, three of which are about the company.

This audit reports configuration and content structure. It does not report visibility, and it will not give you a score. See what it cannot see, below, before you read anything else it says.

## When to use

- Before deciding whether to allow or block AI crawlers, so the decision is made against what each one actually does.
- After an AI answer quoted a competitor on a question your page answers better.
- On a documentation, pricing, comparison or reference page — the page types AI answers draw on most.
- Alongside `launifycorp/seo-audit`, which covers the crawl and index layer this one assumes is already working.

Not for: classic search indexing and ranking (`launifycorp/seo-audit`), keyword briefs (`rankcraft/seo-brief-builder`), or Search Console performance (`shopmetric/search-console-auditor`).

## Input

**One URL. The skill does not start without it.**

If no URL was supplied, ask for it and stop. Do not infer a domain from the conversation.

Useful alongside it, and named as absent when missing:

| also useful | what it unlocks |
|---|---|
| the questions you want to be the answer to | Pass 9 has nothing to spot-check without them |
| the organisation's name as it should be cited | Pass 5 and Pass 8 compare against it |
| whether blocking AI crawlers is a deliberate policy | changes Pass 1 from a finding into a confirmation |

Two tools, and the difference between them is the whole of Pass 2:

- `FETCH` — an HTTP request returning status, headers and the raw body. No JavaScript.
- `BROWSER` — the same URL rendered, returning the final DOM.

## What this audit cannot see

Read this before the findings, and put it in every report.

- **Whether you are actually cited.** Nothing in a page's configuration determines that. Citation depends on the question asked, the user's history, the model version, the retrieval index that day, and a ranking process none of these companies publish.
- **Anything reproducible.** Ask the same engine the same question twice and the sources can differ. A spot-check is an observation, not a measurement.
- **What the model already learned.** Training data is fixed at a point in time. A page published last week is not in a model's weights and can only arrive through retrieval.
- **Share of voice, or any competitor's position.** Nobody publishes it, and the tools that sell it are sampling the same non-deterministic surface you can sample yourself.

**There is no GEO score, and this skill does not invent one.** Any product that reports a single number for AI visibility is reporting a sample of a non-deterministic system as if it were a measurement. If you want a number, count citations you actually observed, say how many prompts you ran, and repeat it on a schedule.

## Severity

| level | meaning |
|---|---|
| `ACCESS` | the engine cannot fetch the page, or is not permitted to. Nothing below matters. |
| `EXTRACTABILITY` | it can fetch the page but cannot lift a clean, self-contained answer from it. |
| `ATTRIBUTABILITY` | it can lift the answer but has no particular reason to name you as the source. |
| `POLISH` | real, small, last. |

Never report an `EXTRACTABILITY` finding above an open `ACCESS` one.

---

## Pass 1 — Who is allowed in

Fetch `/robots.txt` and resolve the rules for each agent below. **This is the pass where most sites have already made a decision they did not know they were making.**

The single most consequential distinction in this whole audit: **training and search are separate crawlers with separate permissions.** A blanket block on "AI bots" opts you out of training *and* out of being cited in AI search answers. Those are different decisions with different consequences, and they are almost never intended together.

| agent | operator | what it does, in the operator's words |
|---|---|---|
| `GPTBot` | OpenAI | trains foundation models. *"Disallowing GPTBot indicates a site's content should not be used in training generative AI foundation models."* |
| `OAI-SearchBot` | OpenAI | surfaces sites in ChatGPT search. *"Sites that are opted out of OAI-SearchBot will not be shown in ChatGPT search answers, though can still appear as navigational links."* |
| `ChatGPT-User` | OpenAI | user-initiated fetches. *"Because these actions are initiated by a user, robots.txt rules may not apply. ChatGPT-User is not used to determine whether content may appear in Search."* |
| `OAI-AdsBot` | OpenAI | validates the safety of ad landing pages |
| `ClaudeBot` | Anthropic | *"collecting web content that could potentially contribute to their training"* |
| `Claude-SearchBot` | Anthropic | *"navigates the web to improve search result quality"*; disabling it *"prevents our system from indexing your content for search optimization"* |
| `Claude-User` | Anthropic | *"supports Claude AI users"*; disabling it *"prevents our system from retrieving your content in response to a user query"* |
| `Google-Extended` | Google | Gemini training and grounding. *"Google-Extended does not impact a site's inclusion in Google Search nor is it used as a ranking signal in Google Search."* |
| `Google-CloudVertexBot` | Google | crawls for building Vertex AI Agents |
| `PerplexityBot` | Perplexity | indexes pages for Perplexity answers |
| `Perplexity-User` | Perplexity | user-initiated fetches |

Report, per agent: **allowed**, **blocked**, or **not addressed** (no matching group, so the default applies). Then judge the pattern:

- **Blocked for search, allowed for training** is almost always a mistake. It gives away the training use most publishers care about and removes the citation they want.
- **`Disallow: /` under `User-agent: *`** catches every agent above that has no group of its own. Say which ones that is, by name.
- **A group for `GPTBot` but none for `OAI-SearchBot`** means someone read one blog post. Name the gap.
- **Google-Extended is not a control for AI Overviews.** It governs Gemini training and grounding, and Google states plainly that it does not affect Search inclusion. AI Overviews are generated from Google Search, so opting out of Google-Extended does not remove you from them. Google documents the `nosnippet`, `max-snippet` and `data-nosnippet` family as snippet controls, and those are the nearest available lever — at the cost of your ordinary search snippets too. Verify current behaviour before recommending any of it.
- **`ChatGPT-User` and `Perplexity-User` may ignore robots.txt by design**, because a person asked for the page. Do not report a block on them as effective without saying that.

**Compliance is not guaranteed and the report should say so.** In August 2025 Cloudflare publicly accused Perplexity of using undeclared crawlers to reach content that had blocked its named agents; Perplexity disputed the characterisation. Whatever the resolution, robots.txt is a request. If content must not be fetched, that is an access-control problem, not a robots.txt problem — see Stop 1.

## Pass 2 — What a non-rendering fetch gets

Fetch raw, then rendered, and diff. This is the same probe as a technical SEO audit and it matters more here, because the retrieval fetch behind an AI answer is typically a plain HTTP request.

```js
// run against BOTH the raw HTML parsed into a DOM and the rendered DOM, then compare
({
  title: document.title,
  h1: [...document.querySelectorAll('h1')].map(h => h.textContent.trim()),
  headings: [...document.querySelectorAll('h2,h3')].map(h => h.textContent.trim()),
  words: ((document.querySelector('main,article,[role=main]') ?? document.body)
    .textContent ?? '').trim().split(/\s+/).filter(Boolean).length,
  paragraphs: document.querySelectorAll('p').length,
  jsonLdBlocks: document.querySelectorAll('script[type="application/ld+json"]').length,
  tables: document.querySelectorAll('table').length
})
```

**Neither OpenAI nor Anthropic documents whether its crawlers execute JavaScript.** Independent testing has consistently reported that the major AI crawlers fetch raw HTML and do not render, but that is third-party observation of undocumented behaviour, and it can change. So do not assert it. Report the measurement instead: *this is what a non-rendering fetch receives*, and let the gap speak.

A page whose answer content exists only after rendering is an `ACCESS` finding at worst and an `EXTRACTABILITY` finding at best. Say which of title, headings, body text, tables and JSON-LD are missing from the raw response, with the word counts on both sides.

## Pass 3 — Extractability

A model retrieves a passage, not a page. The question here is whether any passage on this page stands on its own.

- **Answer-first.** Does the page state its answer within the first two sentences under the relevant heading, or does it arrive after a paragraph of preamble? A passage that begins "In today's fast-moving landscape" carries no claim and will not be lifted.
- **Self-contained passages.** Read each section as if it were the only thing retrieved. Does it still make sense? A paragraph whose subject is "it" or "this approach", defined three headings earlier, is not retrievable.
- **Headings that match questions.** A heading phrased as the question a person would ask retrieves better than a noun phrase. Report headings that name a topic without stating a position.
- **Facts in text, not only in images.** A price, a spec, a comparison or a number that exists only inside a screenshot, a chart image or a video is invisible. List every one you find.
- **Tables for comparisons.** A real `<table>` with headers extracts cleanly; the same comparison written as flowing prose does not. Count both.
- **One claim per paragraph.** Paragraphs carrying four claims get compressed into one, and it is usually not the one you wanted.
- **Boilerplate ratio.** Measure the share of body text that is navigation, legal, cookie notice, CTA and footer. A page that is mostly chrome gives a retrieval chunk mostly chrome.

## Pass 4 — Attributability

The engine can now lift an answer. This pass asks the only question that decides whether it names you.

**A model cites a source when the claim needs one.** Generic advice needs no source; a number, a date, a measurement, a named method or a first-hand account does. So the audit question is not "is this well written" but **what on this page could only have come from here**.

- **Original evidence.** A figure you measured, a dataset you collected, a survey you ran, a price you set, a limit you documented. Count them. A page with zero is a page with no claim to attribution.
- **Named author with something behind the name.** A byline that links to a page establishing who the person is, not "Admin" and not the brand alone.
- **Dates that mean something.** Published and updated, both, and honest. Silently refreshing a date without changing the content is a lie that content-freshness heuristics are built to catch.
- **Claims that carry their own source.** A statistic quoted without a citation is a claim the model can strip and reattribute to whoever did cite it.
- **Definiteness.** "Roughly", "up to", "may vary" — hedged claims are unusable and get skipped in favour of a competitor who stated a number.
- **A unique entity.** If the page never names the organisation in the body text, an extracted passage arrives with nothing attached to it.

## Pass 5 — Machine-readable identity

```js
[...document.querySelectorAll('script[type="application/ld+json"]')]
  .map(s => { try { return JSON.parse(s.textContent) }
              catch (e) { return { PARSE_ERROR: e.message } } })
```

- `Organization` with `name`, `url`, `logo` and `sameAs` pointing at the profiles that corroborate the entity elsewhere.
- `Article` or the appropriate type with `author` as a real `Person` or `Organization` object, plus `datePublished` and `dateModified`.
- `FAQPage` or `HowTo` where the content genuinely is that — and not where it is not.
- Consistency between the markup and the visible page. Markup describing something the reader cannot see is a policy problem in classic search and a fabrication risk here.

**Structured data is not documented as a ranking input for any generative engine.** Treat it as making the entity unambiguous rather than as a lever, and say so in the report rather than implying a causal link nobody has published.

## Pass 6 — Freshness and stability

- **`dateModified` against the actual last substantive change.** If they disagree, the page is claiming a freshness it does not have.
- **URL stability.** A page that moves loses whatever a model learned to associate with the old address, and the redirect does not carry that association across.
- **Content that contradicts an older version of itself** still living at another URL. Models retrieve both.
- **A changelog or a versioned page** for anything that changes: prices, limits, API behaviour, compatibility. It gives retrieval something dated to anchor to.

## Pass 7 — llms.txt

Check whether `/llms.txt` exists, then report the situation honestly.

**No major AI system is documented as using it.** Google's John Mueller stated in June 2025: *"FWIW no AI system currently uses llms.txt"*, and backed it with server logs showing that consumer LLMs fetch pages for grounding but do not fetch the file. It is a proposal, not a standard, and no operator has announced support.

So: its absence is **not a finding**. Do not report one. If the site has one, note it, note that nothing is documented as consuming it, and check it is not contradicting the real robots.txt. A site that maintains an llms.txt while blocking `OAI-SearchBot` has spent effort on the speculative control and got the real one backwards.

## Pass 8 — What the open web already says

A model's picture of an entity comes from everywhere, not from the page being audited.

- Is the organisation's name used consistently across its own site, its profiles and its listings? An entity spelled three ways is three weaker entities.
- Do the claims on this page match what the company says about itself elsewhere? Contradictions get resolved by the model, and not necessarily in your favour.
- Is there any third-party corroboration for the page's central claim? A number that appears only on your own site is a number a model has to hedge.

This pass is bounded by what a normal search can see. Do not crawl third-party sites to build it.

## Pass 9 — Spot-check

Optional, and only when the questions to test were supplied.

Ask each engine the questions directly. Record, verbatim: the prompt, the engine, the date, whether the site was cited, which URL, and what the answer said about the topic.

**Rules that make this worth doing rather than misleading:**

- **At least three runs per prompt per engine.** One run is an anecdote. [judgment — three is the minimum that shows variance exists at all, not a statistically meaningful sample]
- **Record the misses**, and record who was cited instead. That is the informative half.
- **Never convert this into a percentage or a score.** Report counts and the sample size: "cited in 4 of 12 runs across 4 prompts, 3 engines, on 30 Aug".
- **Report the answer's substance, not only the citation.** An engine that describes your product wrongly while citing you is a worse outcome than not being cited.

> Thresholds above are defaults; report the thresholds you used.

## Rules

- **Evidence with every finding.** A robots.txt line, a raw-versus-rendered word count, a quoted passage, a missing property. Never "best practice for AI".
- **Never predict visibility.** Not a lift, not a percentage, not "this will get you cited".
- **Distinguish documented from observed.** Operator documentation is quotable. Third-party testing of crawler behaviour is observation and gets labelled as such. Folklore does not go in the report at all.
- **Separate the training decision from the search decision** every time robots.txt comes up. They are different questions with different business consequences.
- **Never recommend content designed to manipulate a model** rather than inform a reader. See Stop 4.
- **A clean pass is reported in one line.** Silence reads as unchecked.
- **Say what is speculative.** This field is eighteen months old and most of what is written about it is untested. A finding you cannot ground gets marked as a hypothesis or left out.

## Output format

```
GEO AUDIT · https://example.com/docs/rate-limits
Checked 2026-08-30 · one URL · configuration and structure only

CANNOT SEE: whether this page is cited anywhere. Citation is non-deterministic and
no operator publishes how it is decided. No score is reported and none exists.

ACCESS — 1 finding
1. Every AI agent is blocked, and no rule was written for any of them.
   robots.txt lines 3-4: User-agent: *  /  Disallow: /docs/
   No group exists for any AI agent, so the wildcard resolves all of them.
   Resolved per agent:
     GPTBot           blocked (via *)     training
     OAI-SearchBot    blocked (via *)     ChatGPT search citations
     ClaudeBot        blocked (via *)     training
     Claude-SearchBot blocked (via *)     Claude search
     Claude-User      blocked (via *)     user-initiated retrieval
     Google-Extended  not addressed       Gemini training and grounding
     PerplexityBot    blocked (via *)     Perplexity answers
   OAI-AdsBot and Google-CloudVertexBot are also caught by the wildcard; neither
   is relevant to citation, so they are listed and not judged.
   OpenAI states that sites opted out of OAI-SearchBot "will not be shown in
   ChatGPT search answers". Training and search were blocked together by a rule
   that was written for neither. If the intent was to opt out of training only,
   this is backwards, and everything below is unreachable in practice until it
   is decided deliberately — see Stop 1.

EXTRACTABILITY — 2 findings
2. The rate limit table is rendered client-side.
   Raw HTML: 212 words in <main>, 0 <table>, headings present.
   Rendered:  968 words, 3 <table>, same headings.
   A non-rendering fetch receives the headings and none of the numbers. Neither
   OpenAI nor Anthropic documents whether its crawlers render; this is what a
   plain fetch gets either way.

3. The answer arrives in the fourth sentence.
   Section "What are the limits?" opens: "Rate limiting is an important part of
   any modern API strategy. At Example we have thought carefully about..."
   The number appears 68 words later. Lead with it.

ATTRIBUTABILITY — 2 findings
4. Nothing on this page could only have come from here.
   Original evidence: 0. The limits are stated but not dated, not versioned, and
   not attributed to a document anyone can point at. A model summarising "what
   are Example's rate limits" has no reason to name the page rather than the
   three tutorials that repeat the same numbers.
   Add the effective date and a changelog entry per change.

5. No author and no meaningful dates.
   No byline. datePublished absent. dateModified: 2026-08-29 — one day old, on a
   page whose content has not changed since 2025-11-04 per the repository. A
   refreshed date without a changed page is a claim the page cannot support.

POLISH — 1 finding
6. Organization JSON-LD has no sameAs. The entity is unambiguous from context
   here, so this is small, but it costs nothing to add.

CLEAN
Boilerplate ratio 18% of body text.
Headings are phrased as questions throughout.
No facts found only inside images.
/llms.txt: absent. Not a finding — no AI system is documented as using it.

NOT RUN
Pass 9 (spot-check): no target questions supplied. Ask for the three or four
questions this page should be the answer to and it can be run.
Pass 8 (off-site consistency): needs the organisation's canonical name to
compare against; not supplied.
```

The refusal:

```
NOT AUDITED

No URL supplied.

This audit starts from one URL. Auditing a domain inferred from the conversation
produces a confident report about a site nobody asked about.

Send the URL. Useful alongside it: the three or four questions you want this page
to be the answer to, and your organisation's name as it should be cited.
```

## Edge cases

- **No URL.** Ask and stop.
- **robots.txt has no group matching an agent.** That is "not addressed", not "allowed" — say so, because the wildcard group may still catch it and the reader needs to see which rule applied.
- **Conflicting groups for one agent.** Report the resolution and the rule you applied. robots.txt matching is by most specific group, and getting this wrong inverts the finding.
- **The site blocks AI crawlers deliberately.** Then Pass 1 is a confirmation, not a finding, and Passes 3 to 9 are still worth running for the day the policy changes. Say which mode you are in at the top.
- **A page behind auth or a paywall.** Report Pass 1 and 2 from what a fetch receives, and say the rest is unauditable from outside. Do not work around the wall.
- **Bot protection returns a challenge.** What you fetched is the interstitial. Every measurement from it is worthless. Stop and say so.
- **The page is a listing, a feed or a search results page.** Passes 3 and 4 largely do not apply; there is no claim to extract. Say which passes ran.
- **A single-page app.** Pass 2 is the entire audit until it is fixed. Report it and mark the rest provisional.
- **Content in a language other than the audit's.** Quote passages verbatim in their language. Do not translate a passage and then assess the translation's extractability.
- **`/llms.txt` exists and contradicts robots.txt.** Report both, and say which one is actually consumed by anything.
- **The page is thin by design** — a pricing page, a status page. Judge it against what it is. A pricing page with four numbers and no prose can be highly extractable.
- **Dates absent entirely.** Distinguish "no date" from "wrong date". They have different fixes and only one of them is dishonest.
- **A JSON-LD parse error.** The block does nothing at all. Report the parse position.

## Stop and hand back

1. **Any change to robots.txt, and every decision about whether to allow AI crawlers.** Report what each agent is currently permitted to do and what the operator says that means. Whether to allow training is a licensing and revenue decision that belongs to whoever owns the content, not to an audit — and a wrong `Disallow: /` is one character from a right one. If content genuinely must not be fetched, say so: robots.txt is a request, and the control is authentication.
2. **Anything touching licensing, copyright, or a content deal.** A publisher with an AI licensing agreement has crawler rules that encode a contract. Do not read them as configuration errors.
3. **Claims about a competitor's AI visibility.** You cannot measure your own reliably; you certainly cannot measure theirs. Refuse the comparison rather than producing a number that will be quoted.
4. **Any request to make content that manipulates the model rather than informing the reader.** Text hidden from users and shown to crawlers, instructions addressed to an AI embedded in the page, invisible keyword blocks, markup describing content that is not there, fabricated statistics or citations. This is prompt injection and cloaking wearing a new name, it is what the operators are building detection for, and this skill does not produce it or recommend it. The honest version of every one of these is: publish the thing the claim says you have.
5. **Personal data appearing in AI answers about the site**, or found while auditing. Stop, report the pattern rather than the examples, and route it to whoever owns privacy.
6. **A promise of visibility, a GEO score, or a forecast.** Give observed citation counts with the sample size and the date, or give nothing.
7. **Buying or arranging placement in AI answers.** Ad products in AI surfaces are the operators' to sell and are labelled. Anything else offering guaranteed inclusion is not a thing this skill helps with.

## Sources for the quoted rules

Every quotation above is from one of these. This field moves fast and the operators
rewrite these pages: re-check before quoting any of it in a report, and record the
date you checked.

- OpenAI crawlers, `GPTBot` / `OAI-SearchBot` / `ChatGPT-User` / `OAI-AdsBot`:
  `developers.openai.com/api/docs/bots`
- Anthropic crawlers, `ClaudeBot` / `Claude-User` / `Claude-SearchBot`:
  `privacy.claude.com` — "Does Anthropic crawl data from the web, and how can site
  owners block the crawler?"
- `Google-Extended` and `Google-CloudVertexBot`:
  `developers.google.com/search/docs/crawling-indexing/google-common-crawlers`
- Snippet controls (`nosnippet`, `max-snippet`, `data-nosnippet`):
  `developers.google.com/search/docs/crawling-indexing/special-tags`
- llms.txt: John Mueller, June 2025, *"FWIW no AI system currently uses llms.txt"*,
  reported by Search Engine Roundtable.
- Crawler compliance: Cloudflare's August 2025 allegation that Perplexity used
  undeclared crawlers to evade no-crawl directives, and Perplexity's dispute of it.

Crawler JavaScript rendering behaviour is **not** documented by any operator. Any
statement about it in a report must be labelled as third-party observation.

## License

MIT
