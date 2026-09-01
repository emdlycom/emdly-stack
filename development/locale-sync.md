---
name: locale-sync
owner: launifycorp
category: Development
description: Finds a project's translation files, works out which language is the source of truth, and diffs every other locale against it. It reports coverage and gaps first, then offers two ways forward: fill th...
version: v1
license: MIT
updated: 2026-09-01
recommended: false
security_checked: true
url: https://emdly.com/skills/launifycorp/locale-sync
raw: https://emdly.com/raw/launifycorp/locale-sync.md
install: npx @emdly/cli add launifycorp/locale-sync
---

# Locale Sync

Finds a project's translation files, works out which language is the source of truth, and diffs every other locale against it. It reports coverage and gaps first, then offers two ways forward: **fill the gaps** in existing locales so they match 1:1, or **add a whole new language** from scratch. Nothing is written before the user picks one and confirms.

## When to use

- "check my translations", "what's missing in the German locale", "i18n status", "translation coverage"
- "add Polish to the project" / "keep all locales in sync with English"
- Before a release that ships a new language, or after a sprint that added a batch of new UI strings.

Do not use it to translate a single string — that is a direct answer, not an audit.

## Step 1 — Find the translation files

Look for the common layouts before asking the user anything:

| Layout | Example |
|---|---|
| Directory per locale | `locales/en/common.json`, `locales/de/common.json` |
| File per locale | `i18n/en.json`, `i18n/de.json`, `lang/en.yml` |
| gettext | `locale/de/LC_MESSAGES/django.po` + `.pot` template |
| Flutter / ARB | `lib/l10n/app_en.arb` |
| Apple / Android | `en.lproj/Localizable.strings`, `values-de/strings.xml` |
| XLIFF | `translations/messages.de.xlf` |
| Rails | `config/locales/*.yml` |

Search paths like `locales/`, `i18n/`, `lang/`, `translations/`, `messages/`, `public/locales/`. Confirm the framework from config or dependencies (i18next, react-intl, vue-i18n, next-intl, formatjs, gettext, Rails I18n, Flutter intl) — it decides the file format, the plural syntax, and the placeholder syntax.

If nothing is found, say so and stop. Do not invent a structure.

**Namespaces:** when locales are split into several files (`common.json`, `errors.json`, …), diff namespace by namespace. A file missing entirely from a locale is a finding, not an empty result.

## Step 2 — Establish the source language

In order of evidence: an explicit config value (`defaultLocale`, `fallbackLng`, `sourceLanguage`, `i18n.defaultLocale`), then the `.pot` template for gettext, then the locale with the most keys. State which one was chosen and why, and let the user override it. Everything downstream is measured against this one language.

## Step 3 — Diff

Flatten every locale to a set of full key paths (`auth.login.title`) and compare each against the source. Report these categories separately — they need different fixes:

- **MISSING** — key exists in source, absent in this locale.
- **EMPTY** — key exists but the value is `""` or whitespace.
- **UNTRANSLATED** — value is byte-identical to the source. Flag it, never assume it is a bug: brand names, `OK`, `Email` and similar are legitimately identical in many languages.
- **ORPHANED** — key exists in this locale but not in the source. Usually a leftover from a removed feature; deleting it is a separate decision.
- **PLACEHOLDER MISMATCH** — the interpolation tokens differ from the source: `{{count}}`, `{count}`, `%s`, `%1$s`, `:name`, `$t(...)`, or embedded HTML/markup tags. This is the category that causes runtime crashes, so it outranks a merely missing string.
- **PLURAL MISMATCH** — the locale is missing plural forms it needs. This is **not** a per-key copy of the source's forms: CLDR plural categories differ by language. English needs `one`/`other`; Czech needs `one`/`few`/`many`/`other`; Polish `one`/`few`/`many`/`other`; Arabic `zero`/`one`/`two`/`few`/`many`/`other`; Japanese only `other`. Judge each locale against **its own** CLDR categories, and say so in the report.
- **TYPE MISMATCH** — a key is a string in one locale and an object or array in another.

Also count **unused keys** — declared in the source but referenced nowhere in the code (`t('…')`, `<FormattedMessage id>`, `$t('…')`, `trans('…')`). Mark these as *likely* unused: dynamically built keys (`t('errors.' + code)`) cannot be detected statically, so never delete on this signal alone.

## Step 4 — Report, then stop

Show coverage per locale, then the findings grouped by category, most breaking first. Show totals. State plainly that nothing has been written yet, and offer the two modes:

**A — Fill the gaps.** Bring the selected locales to 1:1 with the source. Only MISSING and EMPTY keys are touched. Existing translations are never overwritten.

**B — Add a new language.** Ask for the locale code (BCP 47: `pt-BR`, not `br`), create the full file set from the source structure, and translate every key.

Ask which mode, and for which locales. Do not assume "all".

## Step 5 — Write, after explicit confirmation

Restate the exact locales, the exact key count, and the mode. Wait for a clear yes. Treat an ambiguous reply as no. Then:

1. **Check the working tree is clean** (`git status --short` on the locale paths). If there are uncommitted changes there, say so and let the user decide — this is the only undo that matters.
2. **Preserve the file exactly as it is.** Same indentation, same quote style, same key order as the source, trailing newline, `\n` vs `\r\n`. Insert new keys in the source's position, do not re-sort the file. For `.po`, keep the header, comments, and `#,` flags intact.
3. **Translate with the context in hand.** The key path is meaningful — `auth.errors.expired` is an error message, `nav.home` is a menu label with a length budget. Prefer the UI register of the target language (imperative vs infinitive in menus, formal vs informal address — match whatever the locale already uses; German `Sie` vs `du` is a product decision, so follow the existing file and say which one was found).
4. **Never touch what is inside a placeholder.** `{{userName}}`, `%s`, `{count, plural, ...}`, `<0>…</0>` and HTML tags pass through byte-identical. Only the surrounding words are translated. Word order may move a placeholder within the sentence — that is fine and often required.
5. **Build plural forms from the target language's CLDR categories**, not from the source's. A language with four categories gets four, even though English supplied two.
6. **Mark machine-generated output for review.** Add the format's own marker — `#, fuzzy` in gettext, or a `@@needsReview` note in ARB — or list every touched key in the final report so a human can review them. Never silently pass off generated translations as reviewed.
7. **Write one locale per step** and report as you go, so a failure mid-run leaves a known state.

## Rules

- Never overwrite an existing non-empty translation. Report a suspicious one; do not "improve" it.
- Never delete ORPHANED or unused keys as part of a fill. Deletion is its own confirmed action.
- Never reorder or reformat a file you were only asked to add keys to — it turns a 5-line diff into a 500-line one.
- Never guess a locale code. `cz` is not Czech (`cs`), `jp` is not Japanese (`ja`), `gb` is not English (`en-GB`).
- If the source language itself has empty or duplicate keys, report that first — the source is the contract.
- Say when a translation is uncertain: idioms, legal wording, and marketing copy need a human. List them separately rather than burying them.

## Output format

```
Project: <name>   Framework: <i18next|gettext|...>   Source: <en>
Locales: <n>   Namespaces: <n>   Keys in source: <n>

COVERAGE
  de   412/430   96%
  cs   388/430   90%
  fr   430/430  100%

PLACEHOLDER MISMATCH (2)  — breaks at runtime
  cs  checkout.total        source {{amount}} -> found {{castka}}
  de  errors.retry          source <0>link</0> -> tag missing

PLURAL MISMATCH (1)
  cs  cart.items            has one/other, needs one/few/many/other

MISSING (18)
  de  settings.privacy.title, settings.privacy.body, ... (+16)

EMPTY (4)   UNTRANSLATED (7)   ORPHANED (3)   LIKELY UNUSED (11)

Nothing written yet.
  A) fill missing + empty keys — which locales?
  B) add a new language — which locale code?
```

After writing:

```
Written: de (+18 keys), cs (+42 keys)
Needs human review: <list of generated keys>
Uncertain: checkout.terms — legal wording, review before shipping
Untouched: existing translations, orphaned keys, file formatting
```

## License

MIT
