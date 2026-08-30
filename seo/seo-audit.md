---
name: seo-audit
owner: launifycorp
category: SEO
description: Technical SEO audit that starts from one URL and works outward, ordered by consequence rather than by ease of checking: resolve, crawlability, indexability, rendering, canonicalisation, on-page, structured data, hreflang, speed. Every threshold is quoted from Google or the sitemaps protocol, or labelled as the skill's own judgment.
version: v1
license: MIT
updated: 2026-08-30
recommended: false
security_checked: true
url: https://emdly.com/skills/launifycorp/seo-audit
raw: https://emdly.com/raw/launifycorp/seo-audit.md
install: npx @emdly/cli add launifycorp/seo-audit
---

# SEO audit

Give it a URL. It works outward from that one page to the things that decide whether the page can appear in search at all, and only then to the things that decide how well it does. Most audits are ordered by how easy the check was to run, which is why they open with title tags and bury the `noindex`; this one is ordered by consequence, because a page that cannot be indexed has no title problem worth discussing.

Every finding carries the evidence that produced it, a status code or a header or a line of the file, and a severity that says what it costs. Nothing here is asserted from experience.

## When to use

- A page or a site is not appearing in search and nobody knows which layer is at fault.
- Before a migration, to record what the current state actually is.
- After a launch or a redesign, to catch the `noindex` that shipped with it.
- A recurring technical health check on a template, run on one representative URL per template.
- Auditing someone else's site to understand it — with the crawling limits under Stop and hand back respected.

Not for: keyword research or content briefs (`rankcraft/seo-brief-builder`), performance data from Search Console (`shopmetric/search-console-auditor`), or visual and UX quality (`launifycorp/design-review`).

## Input

**One URL. This is the entry point and the skill does not start without it.**

If no URL was supplied, ask for exactly that and stop. Do not audit a domain you inferred from context, from a company name, or from a previous message. Auditing the wrong host produces a confident report about a site nobody asked about.

Ask for these alongside the URL only when the answer changes what you check:

| also useful | what it unlocks |
|---|---|
| scope: this page, this template, or the site | whether Pass 5 and Pass 2 run against a crawl or a single URL |
| the market and language targeted | Pass 8 is meaningless without knowing the intended set |
| whether the site is behind auth, staging, or bot protection | see Edge cases before you start firing requests |
| a representative URL per template | one product, one category, one article beats fifty products |

Two tools, and the difference between them is a finding in its own right:

- `FETCH` — an HTTP request that returns status, headers and the raw response body. No JavaScript.
- `BROWSER` — the same URL rendered, returning the final DOM.

Where the skill says "raw", it means `FETCH`. Where it says "rendered", it means `BROWSER`. If only one is available, say so in the report and mark Pass 4 as not run rather than guessing.

## What this audit cannot see

State this in the report, every time, because the absence shapes what the findings are worth.

- **No Search Console.** No impressions, no queries, no coverage report, no manual actions. This audit sees the site as a crawler would on this request, not what Google actually did with it.
- **No server logs.** Real crawl frequency and real bot behaviour are invisible.
- **No backlink data.** Nothing here speaks to authority, link profile, or off-site anything.
- **No ranking data.** This audit never says a page will rank, or how it compares to a competitor.
- **One point in time, from one location.** A CDN may serve something else elsewhere.

An audit that quietly omits this reads as more complete than it is.

## Severity

Four levels, defined by consequence, not by effort. The order is the deliverable.

| level | meaning |
|---|---|
| `INDEXABILITY` | the page cannot be crawled, cannot be indexed, or resolves to something other than what was asked for. Nothing below matters until this is clear. |
| `RELEVANCE` | the page can be indexed but describes itself wrongly, competes with itself, or hides its content from the crawler. |
| `EFFICIENCY` | crawl and duplication waste. Only a finding when the site clears the thresholds in Pass 5 — on a small site this level is usually empty and should say so. |
| `POLISH` | real, small, and last. |

**Never report a `RELEVANCE` finding above an open `INDEXABILITY` one.** A reader who fixes the title of a page that is disallowed in robots.txt has been actively misled.

---

## Pass 1 — Resolve

Before anything is measured, establish what the URL actually is. This pass finds more real problems than any other and almost every audit skips it.

```bash
# every hop, with its status. -o /dev/null would discard exactly what we need here,
# and %{http_code} reports only the final response, so read the header stream.
curl -sSIL "$URL" | grep -iE '^(HTTP/|location:)'

# the four host and protocol variants: only the endpoint matters, so -w is fine here
for v in "http://example.com" "http://www.example.com" \
         "https://example.com" "https://www.example.com"; do
  printf '%s -> ' "$v"
  curl -sSIL -o /dev/null -w '%{http_code} %{url_effective}\n' "$v"
done

# trailing slash: both forms returning 200 means two URLs for one page
for u in "${URL%/}" "${URL%/}/"; do
  printf '%s -> ' "$u"
  curl -sSI -o /dev/null -w '%{http_code}\n' "$u"    # no -L: we want the first response
done
```

Record and judge:

- **The redirect chain, every hop with its status.** A `302` where a `301` was meant is a finding. A chain that ends where it started is a loop and is `INDEXABILITY`. On chain length Google publishes no threshold: this skill treats two hops as tolerable and three as worth reporting, because each hop is a round trip for every crawler and every visitor. [house rule]
- **Variant convergence.** All four host and protocol variants should end at one URL. Two that both return `200` are two pages with the same content, and the site is competing with itself.
- **The final status.** A `200` that renders "page not found" is a soft 404: it will keep being crawled and it will never rank. Check the rendered text, not only the code.
- **Whether the URL you were given is the one the page declares canonical.** If not, say so here, in Pass 1, and audit the canonical as well. Auditing a URL the site itself does not consider primary produces findings about a page that is not in the index.

## Pass 2 — Crawlability

Fetch `/robots.txt` from the resolved host. Its **status code matters as much as its contents**, and this is where the non-obvious failures live.

Google's own specification, quoted:

- **A 4xx, except 429, is not a restriction.** *"Google's crawlers treat all 4xx errors, except 429, as if a valid robots.txt file didn't exist. This means that Google assumes that there are no crawl restrictions."* So a missing robots.txt is safe, and a 403 on robots.txt is also read as "no rules".
- **A 5xx behaves in two opposite ways, and which one depends on the cache.** With a cached copy: Google pauses crawling for the first 12 hours, uses the last cached version for the next 30 days, and past 30 days treats the site as having no robots.txt. Without one, the spec is explicit — *"If there's no cached version available, Google assumes there's no crawl restrictions."* So on an established site a 5xx is a crawl stop and an `INDEXABILITY` finding; on a host Google has never successfully fetched robots.txt from — a new site, a new subdomain, a migration cutover — it is the opposite, and everything is open. Say which case you are in, or say you cannot tell.
- **Size limit:** *"Google enforces a robots.txt file size limit of 500 kibibytes (KiB). Content which is after the maximum file size is ignored."* Check the byte size. A rule below the cut is not a rule.
- **Caching:** *"Google generally caches the contents of robots.txt file for up to 24 hours."* A fix is not instant, and a report should not imply it is.
- **Supported fields only:** *"Google supports the following fields (other fields such as crawl-delay aren't supported): user-agent, allow, disallow, sitemap."* A `crawl-delay` line is not doing anything for Google. `noindex` in robots.txt is not a supported field either.

**The trap worth its own paragraph.** A URL disallowed in robots.txt cannot have its page-level `noindex` read, because the crawler never fetches the page to see it. Google states the consequence: *"Google can't index the content of pages which are disallowed for crawling, but it may still index the URL and show it in search results without a snippet."* So `Disallow` plus `noindex` is not belt and braces — it is the belt preventing the braces from working, and the URL can persist in results. Flag this combination as `INDEXABILITY` whenever both are present.

**Sitemaps.** Take them from the robots.txt `Sitemap:` lines and from `/sitemap.xml`. The sitemaps.org protocol, quoted: *"each Sitemap file that you provide must have no more than 50,000 URLs and must be no larger than 50MB (52,428,800 bytes)"*, index files *"may not list more than 50,000 Sitemaps"* under the same size cap, and *"all URLs in a Sitemap must be from a single host"*. Then check what most audits do not: does the sitemap list URLs that are disallowed, non-canonical, redirecting, or returning anything but `200`? A sitemap is a statement that these URLs are the ones worth having, and contradictions between it and the rest of the site are findings.

## Pass 3 — Indexability

Three places can carry an index directive and they are frequently inconsistent. Check all three; auditors routinely check one.

```bash
# the header — the one people forget. -L matters: without it you read the headers of
# the redirect, not of the page that actually gets indexed.
curl -sSIL "$URL" | grep -i 'x-robots-tag'
# -I sends HEAD; a few servers set the header only on GET, so confirm on a miss:
curl -sSL -D - -o /dev/null "$URL" | grep -i 'x-robots-tag'
```

```js
// the meta tag, in the rendered DOM
[...document.querySelectorAll('meta[name="robots"], meta[name="googlebot"]')]
  .map(m => ({ name: m.name, content: m.content }))
```

- **`X-Robots-Tag` in the HTTP header** applies to any file type and is invisible in the page source. It is the single most common cause of "the page looks fine but is not indexed".
- **`<meta name="robots">` in the rendered DOM**, plus any `googlebot`-specific tag. These are **combined, not overridden**: Google's specification says that where several crawlers are named with different rules, *"the search engine will use the sum of the negative rules"*, and gives `robots: nofollow` alongside `googlebot: noindex` as producing `noindex, nofollow` for Googlebot. So `robots: noindex` plus `googlebot: index` is still noindex. Report the union you computed.
- **Conflicts resolve to the most restrictive.** Google states it for rules within the tag — *"In the case of conflicting robots rules, the more restrictive rule applies."* Header-versus-meta resolution is not documented; treating the union of the negative rules as authoritative there is this skill's extrapolation from the same principle, and the report should say so rather than attribute it to Google. Report both values and the resolution, never only the one you found first.
- **A `noindex` injected by JavaScript** is a real noindex once the page is rendered. Compare raw and rendered here, not only in Pass 4.

An `INDEXABILITY` finding here ends the useful part of the audit for that URL. Say so plainly and continue the remaining passes marked as provisional.

## Pass 4 — Rendering

Fetch the page raw, then rendered, and compare. The gap between them is what a crawler has to execute JavaScript to see.

```js
// in the rendered DOM: what is actually there
({
  h1: [...document.querySelectorAll('h1')].map(h => h.textContent.trim()),
  title: document.title,
  canonical: document.querySelector('link[rel=canonical]')?.href ?? null,
  robots: document.querySelector('meta[name=robots]')?.content ?? null,
  // filter(Boolean): ''.split(/\s+/) is [''], which would report an empty page as 1 word
  wordsInMain: ((document.querySelector('main,article,[role=main]') ?? document.body)
    .innerText ?? '').trim().split(/\s+/).filter(Boolean).length,
  // only real navigable links count; mailto:, tel: and javascript: are not links to a crawler
  realLinks: [...document.querySelectorAll('a[href]')]
    .filter(a => a.protocol === 'http:' || a.protocol === 'https:').length,
  // inline handlers only — see the caveat below, this undercounts framework-bound elements
  inlineJsClickables: [...document.querySelectorAll('[onclick],[role=link]')]
    .filter(e => !e.getAttribute('href')).length
})
```

If `BROWSER` is a real headless browser, `innerText` is right because it respects visibility. If it is a DOM parser such as jsdom, `innerText` is undefined and the probe throws — fall back to `textContent` and say in the report that hidden text is included in the count.

Compare against the raw HTML for the same four fields plus the word count. Report:

- **Primary content present only after rendering.** Not fatal, and not free. Say which of title, H1, canonical, robots and body content are missing from the raw response.
- **Navigation that is not links.** An element with a click handler is not a link and passes nothing. The probe above catches inline `onclick` and `role="link"` only; navigation bound with `addEventListener`, which is how every current framework does it, matches neither and reports zero. Treat the count as a floor, not a measurement, and check the rendered `<nav>` by eye for clickable elements with no `<a href>` under them.
- **A canonical or robots value that differs between raw and rendered.** This is a genuine conflict and belongs in Pass 3 as well.

## Pass 5 — Canonicalisation, duplication, crawl waste

**Canonical is a signal, not an instruction.** Google calls `rel="canonical"` *"a strong signal that the specified URL should become canonical"* and says that without one *"Google will identify which version of the URL is objectively the best version to show to users in Search."* It also prefers HTTPS over equivalent HTTP, and prefers URLs that are part of hreflang clusters. So a canonical pointing somewhere implausible will be overridden, and the report should say "declared" rather than "set".

Google recommends a self-referential canonical on the canonical page itself. Check for one, then check the failure modes:

- canonical → a URL that redirects
- canonical → a URL that returns 404 or 5xx
- canonical → a `noindex` page, which asks Google to consolidate onto a page that must not be indexed
- canonical → a different-language version, which is a job for hreflang, not canonical
- canonical present on paginated pages pointing all of them at page one
- more than one canonical element, which is self-cancelling

**Crawl waste has a threshold, and below it this level is empty.** Google names the sites that need to think about crawl budget: *"Large sites (1 million+ unique pages) with content that changes moderately often (once a week)"*, *"Medium or larger sites (10,000+ unique pages) with very rapidly changing content (daily)"*, and sites with a large share of URLs reported as `Discovered - currently not indexed`. Google adds that *"The numbers given here are a rough estimate to help you classify your site."* If the audited site is nowhere near them, say that crawl budget is not a live concern here and do not manufacture `EFFICIENCY` findings. Google does say to *"consolidate duplicate content"* and to *"eliminate soft 404 errors"*, which *"will continue to be crawled, and waste your budget"* — those two apply at any size.

Above the threshold, look at parameter URLs, faceted navigation, session identifiers, sort orders and endless pagination.

## Pass 6 — On-page

Only meaningful once Passes 1 to 3 are clear.

```js
({
  title: document.title,
  titleLen: document.title.length,
  metaDesc: document.querySelector('meta[name=description]')?.content ?? null,
  headings: [...document.querySelectorAll('h1,h2,h3,h4,h5,h6')]
    .map(h => h.tagName + ' ' + h.textContent.trim().slice(0, 60)),
  // currentSrc is '' until the image loads, which is exactly the lazy-loaded set
  imgs: [...document.images].map(i => ({
    src: (i.currentSrc || i.getAttribute('src') || '').split('/').pop(),
    alt: i.alt, hasAlt: i.hasAttribute('alt') })),
  // bucket only http(s) links: a mailto: has hostname '' and would count as external
  links: [...document.querySelectorAll('a[href]')]
    .filter(a => a.protocol === 'http:' || a.protocol === 'https:')
    .reduce((acc, a) => { acc[a.hostname === location.hostname ? 'internal' : 'external']++;
      return acc }, { internal: 0, external: 0 })
})
```

- **Titles have no published character limit and Google rewrites them.** Google states there is no limit on length and that the title link *"is truncated in Google Search results as needed, typically to fit the device width"*. Truncation is by rendered pixel width, not by characters, so a count is a proxy that fails early on capitals and wide letters. Report the character count, label it a proxy, and never call a rewritten title a penalty. `rankcraft/seo-brief-builder` carries the fuller version of this rule with the pixel anchor; defer to it rather than restating it differently.
- **Heading order that does not skip levels.** On the count of `h1` elements Google has said repeatedly that more than one is fine and the HTML specification permits it, so a single `h1` is a convention this skill reports against, not a rule it enforces. [house rule] Report the outline and let the reader judge.
- **A missing `alt` attribute and an empty `alt=""` are different things.** Empty is correct for decorative images. Report them separately or the number is noise.
- **Is this page linked from anywhere on the site?** A page reachable only from the sitemap is a finding, and it is one a single-URL audit can only raise as a question. Say which.
- **Meta description does not rank the page.** It affects the snippet. Report it as such, and never as a ranking factor.

## Pass 7 — Structured data

```js
[...document.querySelectorAll('script[type="application/ld+json"]')]
  .map(s => { try { return JSON.parse(s.textContent) } catch (e) { return { PARSE_ERROR: e.message } } })
```

- Report every `@type` found, and whether the required properties for that type are present. Missing recommended properties are `POLISH`; missing required properties mean the markup does not qualify, which is `RELEVANCE`.
- **Markup that describes something not visible on the page is a policy problem, not an optimisation.** A review rating in JSON-LD with no reviews on the page is the clearest example. Flag it as `RELEVANCE` and say plainly that it risks a manual action.
- A `PARSE_ERROR` means the block does nothing at all. That is a finding, and an easy one to miss by eye.
- Do not validate by intuition. Name the type and the properties you checked, so the reader can run the same check.

## Pass 8 — International

Only when the site targets more than one language or region. Skip it explicitly otherwise, with one line saying so.

- Every version must list **itself and all the others**. Google: *"Each language version must list itself as well as all other language versions."* A one-way annotation is ignored.
- `x-default` for visitors matching nothing. Google calls it *"recommended for specifying the fallback page for users whose language settings don't match any of your site's localized versions."*
- Google states it *"doesn't use hreflang or the HTML lang attribute to detect the language of a page"* — it uses its own algorithms. So hreflang organises a set of known versions; it does not tell Google what language a page is in, and a report should not imply otherwise.
- Check that hreflang targets return `200`, are canonical, and are not `noindex`. An hreflang set pointing at redirects is a common and invisible failure.

## Pass 9 — Speed

Core Web Vitals, at the **75th percentile of real loads**, split by device: LCP at or under 2.5 s, INP at or under 200 ms, CLS at or under 0.1.

One automated load is a single sample from one machine on one connection. It is a lab number and it must be labelled one. Report it, name it as lab, and say that the field distribution is what Google evaluates. INP in particular needs real interaction; if nothing was clicked, report it as not measured rather than estimating it.

## Rules

- **Evidence with every finding.** A status code, a header line, a file line number, a quoted attribute. A finding whose evidence is "best practice" does not go in the report.
- **Order by severity, then by breadth.** A `RELEVANCE` issue on every template outranks one on a single page.
- **Say which URL each finding is about.** In a template audit this is the difference between a site-wide problem and one bad page.
- **Never predict rankings.** Not a position, not traffic, not a percentage uplift. This audit sees the site's configuration, not the index and not the competition.
- **Distinguish "declared" from "in effect".** The site declares a canonical; Google decides one. The site declares hreflang; Google may ignore an unreciprocated one.
- **Every threshold is Google's, quoted, or labelled as this skill's judgment.** No number arrives without a provenance.
- **A clean pass is reported as a clean pass**, in one line. Silence reads as unchecked.

> Thresholds above are defaults; report the thresholds you used.

## Output format

```
SEO AUDIT · https://example.com/products/merino-tee
Resolved 2026-08-30 · one URL, product template · lab data only

NOT SEEN: Search Console, server logs, backlinks, rankings. One location, one load.

INDEXABILITY — 2 findings, both open. Ordered site-wide first.
1. robots.txt is intermittently 503.
   5 requests over 4 minutes: 503, 503, 200, 503, 200.
   Google's spec: with a cached copy, a 5xx pauses crawling for 12 hours and
   falls back to the last cached version for 30 days. This host has been indexed
   for years, so the cached branch is the one that applies. Site-wide, and it
   outranks every page-level finding here.
   The rules quoted in finding 2 are from one of the successful fetches.

2. The page is disallowed and noindexed at the same time.
   robots.txt line 14 (from the 200 response): Disallow: /products/
   Rendered <meta name="robots" content="noindex,follow">
   Google states it "can't index the content of pages which are disallowed for
   crawling, but it may still index the URL and show it in search results
   without a snippet" — so the crawler never fetches the page and never reads
   the noindex, and the URL can persist. The two directives cancel: pick one.
   If the page should be out of the index, remove the Disallow and keep the
   noindex until it drops out.
   The sitemap contradicts both: it lists 340 URLs under /products/, which is
   the path robots.txt disallows. The site is submitting URLs it also blocks.
   Every finding below is provisional while both of these stand.

RELEVANCE — 3 findings
3. Canonical points at a redirect.
   <link rel="canonical" href="https://example.com/products/merino-t-shirt">
   That URL -> 301 -> /products/merino-tee (this page).
   Canonical is "a strong signal", not an instruction, so Google will likely
   resolve it anyway — but the site is signalling one thing and serving another.
   Point the canonical at the final URL.

4. Primary content is rendered client-side.
   Raw HTML: 84 words in <main>, no <h1>, no canonical.
   Rendered DOM: 611 words, h1 "Merino T-Shirt", canonical present.
   Title and description are in the raw response; everything else is not.

5. JSON-LD declares aggregateRating with no reviews on the page.
   @type Product · aggregateRating.ratingValue 4.8 · reviewCount 212
   No review content in the rendered DOM. Markup describing content that is not
   visible is a policy problem, not an optimisation. Remove it or render the
   reviews.

EFFICIENCY — none
Site is well below the sizes Google names for crawl budget (1M+ pages changing
weekly, or 10,000+ changing daily). Not a live concern here.

POLISH — 2 findings
6. Heading order skips h2 to h4 in the specification block.
7. 4 of 22 images have no alt attribute. A further 9 have alt="" — correct for
   decorative images, listed separately and not counted as a defect.

RESOLVE
Redirect chain: 1 hop, 301, http -> https. All four host variants converge on
https://example.com. No trailing-slash duplicate: /products/merino-tee/ -> 301.
The URL supplied is NOT the one this page declares canonical — see finding 3.
Both were audited; the declared canonical redirects back to this URL.

NOT APPLICABLE
Pass 8 (international) skipped: one language, no hreflang annotations present,
and no second market stated.
Internal linking: this is a single-URL audit, so whether anything on the site
links to this page could not be established. Ask for a crawl or the sitemap
context if that matters.

CLEAN
X-Robots-Tag: absent on both the redirect and the final response.
Sitemap: 1,204 URLs, 2.1 MB uncompressed, single host, all 20 sampled URLs
return 200. Not clean on the disallow overlap — counted under finding 2.

LAB SPEED — not field data, so not raised as a finding
LCP 3.1 s · CLS 0.04 · INP not measured (no interaction fired).
LCP is over the 2.5 s threshold on this single load. That is one sample from one
machine on one connection, and Google evaluates the 75th percentile of real
loads, so it is reported and not ranked. If the field data agrees, it is a
RELEVANCE finding; this audit cannot tell you whether it does.
```

Thin, because the first pass ended it:

```
SEO AUDIT · https://staging.example.com/
Resolved 2026-08-30 · one URL

INDEXABILITY — 1 finding, terminal
1. The whole host is disallowed.
   robots.txt line 2: Disallow: /  (User-agent: *)
   X-Robots-Tag: noindex, nofollow
   HTTP 401 on every URL sampled.
   This is a staging host and it is configured like one. Nothing below
   indexability is worth auditing here.

NOT AUDITED
Passes 4 to 9 were not run. Auditing on-page quality on a host that is
deliberately closed produces findings nobody will act on.

If this URL was supplied by mistake, the production host is the one to audit.
If it was not, this host looks correctly configured to stay out of the index.
```

The refusal:

```
NOT AUDITED

No URL supplied.

This audit starts from one URL and works outward from it. I will not infer a
domain from the conversation: auditing the wrong host produces a confident
report about a site nobody asked about, and every finding in it would be true
of something and useless to you.

Send the URL. Useful alongside it, but not required: whether you want this one
page, one URL per template, or the site; the markets and languages it targets;
and whether it sits behind auth, staging or bot protection.
```

## Edge cases

- **No URL.** Ask and stop. This is the one input the skill cannot substitute for.
- **The URL is not a page.** A PDF, an image, a feed. Passes 1 to 3 still apply — `X-Robots-Tag` works on any file type — and Passes 4, 6 and 7 do not. Say which ran.
- **A redirect chain that leaves the host.** Audit where it lands, and say the URL you were given is not on the site you ended up auditing.
- **A 401 or 403 on the URL itself.** Report Passes 1 to 3 from the headers and stop. Do not attempt to get around authentication.
- **Bot protection, a challenge page, or a CAPTCHA.** What you fetched is the challenge, not the site. Every measurement from that response is worthless — say so and stop rather than reporting on the interstitial.
- **A single-page app with client-side routes.** Check that each route has a real URL that returns `200` when requested directly, not only when navigated to in the app. A route that only exists after a client-side transition is not a page.
- **Infinite URL space** — calendars, filters, search results. Sample, do not crawl. Report the pattern and an estimate of the space, not a list.
- **A sitemap over the limits.** Report the count and the byte size against 50,000 URLs and 52,428,800 bytes. The size cap is on the **uncompressed** file, so measure the decompressed bytes of a `.xml.gz`, not what is on disk. Check whether the file is an index rather than a sitemap; it has its own limits.
- **A robots.txt that is not a robots.txt.** A `200` serving an HTML 404 page, a login form, or a JavaScript app shell. Every line parses as an unrecognised field, so the file reads as empty and the site reads as unrestricted. Check the content type and the first bytes before parsing, and report a soft robots.txt as its own finding rather than as "no rules".
- **A sitemap that is not well-formed XML.** A parse error means the whole file is discarded, not the bad entry. Report the parse position, and never partially parse it into findings.
- **An empty robots.txt.** Zero bytes and a `200` is valid and means allow-all. Say that explicitly, because an empty file and a missing file look identical in a report and only one of them is deliberate.
- **An empty sitemap, or a `200` with an empty body on the audited URL.** Report the byte count. A zero-length `200` is a different problem from a 404 and gets diagnosed differently.
- **A sitemap listing URLs on another host.** The protocol requires a single host. Report it, because those URLs are being ignored.
- **The URL differs from the page's declared canonical.** Audit both, and lead with the discrepancy.
- **The site geolocates or serves by language automatically.** Note that everything measured came from one location and one `Accept-Language`, and that a visitor elsewhere may get a different page.
- **An IDN or punycode host.** Record both forms. hreflang and canonical comparisons must be made on the same form or they will look mismatched when they are not.
- **A staging or preview host.** See the second example. Do not audit it as if it were production, and see Stop 3.
- **Rendering unavailable.** Pass 4 is not run and is reported as not run. Do not infer client-side rendering from the absence of content in the raw HTML — say the raw response lacked it and that you could not confirm what rendering adds.
- **A site so slow the fetch times out.** That is a finding, at `INDEXABILITY`, before it is an inconvenience.

## Stop and hand back

1. **Any change to robots.txt, a canonical, a redirect, or an index directive on a live site.** Propose the change with the evidence, and hand it to whoever owns deployment. A wrong `Disallow: /` removes a site from search, and it is one character away from a right one.
2. **A `noindex` you would recommend removing.** It is often deliberate — staging, thin pages held back on purpose, a legal or contractual requirement, content excluded during a migration. Report it as present and intentional-until-confirmed, and ask before treating it as a defect.
3. **Staging, internal, or pre-release URLs found indexed.** That is an exposure before it is an SEO finding. Route it to whoever owns the site, and do not list the URLs in a report that circulates more widely than they should.
4. **Personal data visible in indexed URLs or titles** — names, order numbers, e-mail addresses, tokens. Stop, report the pattern rather than the examples, and route it to whoever owns privacy.
5. **A suspected manual action or a ranking collapse.** This audit cannot see manual actions; they live in Search Console. Say that plainly rather than diagnosing a penalty from configuration, which is guessing dressed as analysis.
6. **A migration or a redirect map.** Auditing the current state is in scope. Designing the mapping that moves a site is a project with a rollback plan, and it is not this.
7. **Crawling at any volume, or crawling a site you were not asked to audit.** One URL and its immediate references is a fetch. A crawl needs the site owner's agreement, a rate limit and a declared user agent. Never crawl a competitor's site beyond the pages a visitor would open.

## Sources for the quoted rules

Every quotation above comes from one of these. Re-check them before quoting in a
report: Google moves and rewrites these pages, and the robots.txt specification
has already moved once.

- robots.txt parsing, size, caching, 4xx and 5xx handling, supported fields:
  `developers.google.com/crawling/docs/robots-txt/robots-txt-spec`
- `noindex` and disallowed URLs in results:
  `developers.google.com/search/docs/crawling-indexing/block-indexing`
- robots meta tag, combined rules, most-restrictive resolution:
  `developers.google.com/search/docs/crawling-indexing/robots-meta-tag`
- Canonicalisation as a signal, self-referential canonicals, HTTPS and hreflang
  preferences: `developers.google.com/search/docs/crawling-indexing/consolidate-duplicate-urls`
- Crawl budget thresholds, duplicate consolidation, soft 404s:
  `developers.google.com/search/docs/crawling-indexing/large-site-managing-crawl-budget`
- hreflang reciprocity, `x-default`, language detection:
  `developers.google.com/search/docs/specialty/international/localized-versions`
- Title length and truncation:
  `developers.google.com/search/docs/appearance/title-link`
- Sitemap URL count, size and single-host rule: `sitemaps.org/protocol.html`
- Core Web Vitals thresholds at the 75th percentile: `web.dev/articles/vitals`

## License

MIT
