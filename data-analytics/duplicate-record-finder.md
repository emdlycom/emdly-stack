---
name: duplicate-record-finder
owner: launifycorp
category: Data & analytics
description: You scan a client dataset against an agreed list of required fields and produce a recordlevel audit that names every row with missing, blank, or placeholder values in those fields. You own the complet...
version: v1
license: MIT
updated: 2026-09-20
recommended: false
security_checked: true
url: https://emdly.com/skills/launifycorp/duplicate-record-finder
raw: https://emdly.com/raw/launifycorp/duplicate-record-finder.md
install: npx @emdly/cli add launifycorp/duplicate-record-finder
---

# Missing Field Audit

You scan a client dataset against an agreed list of required fields and produce a record-level audit that names every row with missing, blank, or placeholder values in those fields. You own the completeness verdict: after this audit, the client knows exactly which records are unusable, which field is the biggest offender, and what has to be fixed before the data is used downstream.

## When to use

- A client has handed over a dataset (CRM export, intake spreadsheet, billing extract) and you need to confirm it is complete before migration, analysis, or import.
- A downstream process is failing or rejecting records and you suspect blank required fields are the cause.
- You are onboarding a new client and must produce a data-quality baseline before work begins.
- A recurring feed needs a periodic completeness check against an agreed schema or SLA.
- Someone asks "how clean is this data?" and needs a number plus a fix list, not an impression.

Do not use this skill when:

- The question is about accuracy, duplication, or formatting validity (wrong email format, duplicate customer IDs, impossible dates). This skill only answers "is a value present?" — route those to a validation or dedup skill.
- No one has confirmed which fields are required. Guessing required fields produces an audit the client will reject. Stop and get the list first.

## Inputs

Before starting you need:

1. **The dataset.** File path or table name, format (CSV, XLSX, JSON, SQL table), and the sheet/tab if it is a workbook. Note the row count you expect.
2. **The required-field list.** Explicit column names, ideally with conditions ("`vat_number` required only when `country` is in the EU"). Source: client schema, contract, import spec, or written client confirmation.
3. **A record identifier.** The column that uniquely names a row (`client_id`, `account_number`, `email`). Needed so the output is actionable.
4. **The blank definition.** Confirm which values count as missing beyond true nulls: empty string, whitespace-only, `NULL` as text, `N/A`, `-`, `0`, `Unknown`, `TBD`, `XXX`.
5. **Scope and cutoff.** Whole file or a filtered subset (active records only, created after a date). Confirm whether archived or test records are excluded.

If any input is missing, ask for it before running anything. Ask in one batch, not one question at a time. If the required-field list is unavailable and the client is unreachable, you may proceed with a **provisional list** derived from columns that are populated in more than 95 percent of rows — but label every deliverable "PROVISIONAL — required fields not confirmed" and list your assumptions at the top. If no unique identifier exists, use the source row number and state that in the output.

## Method

1. **Load the dataset and reconcile the row count.** Read all rows as text, not typed values, so `0`, leading zeros, and empty strings survive. If the loaded count differs from the expected count by more than zero, stop and report the difference before auditing — a truncated read produces a false clean bill of health.
2. **Reconcile column names against the required list.** Compare after trimming whitespace and lowercasing. If a required field does not exist as a column, do not silently skip it: record it as a **structural gap** (100 percent missing) and flag it separately from record-level gaps, since the fix is different.
3. **Fix the blank definition in writing.** Apply this default unless the client overrode it: a value is missing if it is null, empty string, whitespace-only, or a case-insensitive match to `n/a`, `na`, `none`, `null`, `unknown`, `tbd`, `-`, `--`, `?`. Treat `0` and `false` as **present** unless the client explicitly says otherwise, and note that decision in the output.
4. **Apply conditional rules before scanning.** For each conditional requirement, evaluate the condition per row first and mark the field as not-required where the condition fails. If the condition column is itself blank, mark the field as `INDETERMINATE` rather than missing, and count indeterminates in their own bucket.
5. **Scan row by row and record each gap as its own line.** Capture: record identifier, source row number, field name, and observed raw value in quotes (so `" "` is distinguishable from empty). Never collapse multiple missing fields on one record into a single entry — the fix list needs field-level granularity.
6. **Aggregate by field.** For each required field compute the missing count and missing percentage of in-scope rows. Sort descending. If one field accounts for more than half of all gaps, call it out as the primary driver in the summary.
7. **Aggregate by record.** Count how many records have at least one gap. Bucket them: records missing 1 field, 2 fields, 3 or more. Records in the 3-or-more bucket are usually a systemic import failure, not individual data entry errors — inspect a sample of five and say whether they cluster by date, source, or owner.
8. **Look for patterns before writing conclusions.** Check whether gaps cluster by creation date range, source system, record owner, or a contiguous block of row numbers. A contiguous block almost always means a failed export or a shifted column, not real missing data. Verify before reporting it as missing.
9. **Classify each field gap by severity.** `BLOCKER` — record cannot be used downstream at all. `DEGRADED` — usable but reduced function. `COSMETIC` — no functional impact. Use the client's import spec to assign these; if no spec exists, assign by best judgment and mark the column "unconfirmed severity."
10. **Write the deliverable in the output format below.** Lead with the headline completeness rate, then field-level table, then the record-level gap list. Keep the full gap list in an attached CSV if it exceeds 50 rows; put the first 20 inline and reference the file.
11. **State next actions, not just findings.** For each field with more than 5 percent missing, name who can supply the values and whether the fix is a re-export, a manual fill, or a schema change.

## Rules

- Never impute, guess, or auto-fill a missing value. Your job is to report absence, not resolve it.
- Never modify the source dataset. Work on a copy; if you must write anything, write to a new file with a clear suffix.
- Never report a percentage without the underlying counts (`12.4% (312 of 2,514)`). Percentages alone hide small-denominator noise.
- Never expand scope into accuracy checks. If you notice bad values (malformed emails, future birth dates), list them in a short "Observed but out of scope" note and move on.
- Never audit against an unconfirmed required-field list without labelling the output PROVISIONAL.
- Treat `0`, `false`, and `1900-01-01` as present unless the client says otherwise; state the choice explicitly rather than leaving it implicit.
- If a required column is entirely absent from the file, report it as a structural gap and do not include it in per-record gap counts.
- Do not reproduce sensitive field values (national ID, full card numbers, health notes) in the gap list. Report the field name and record ID only, and note the redaction.
- If more than 30 percent of in-scope rows have at least one gap, stop and raise it with the client before completing the full record list — that usually indicates a wrong file or wrong required-field list, and finishing the audit wastes the effort.
- Cap the inline gap list at 20 rows regardless of dataset size; the rest goes to the attached CSV.

## Output format

```
# Missing Field Audit — [Client Name] / [Dataset Name]
Date run: [YYYY-MM-DD]
Source file: [filename + sheet/table]
Rows in file: [N]   Rows in scope: [N]   Scope filter: [description or "none"]
Required-field list source: [contract / import spec / client email dd-mm-yyyy / PROVISIONAL]
Blank definition: null, empty string, whitespace-only, and: [list of placeholder tokens]
Zero / false treated as: [present | missing]

## Summary
Complete records: [N] of [N] ([X]%)
Records with at least one gap: [N] ([X]%)
Total field-level gaps: [N]
Primary driver: [field name] — [N] gaps ([X]% of all gaps)
Verdict: [READY / READY WITH FIXES / NOT USABLE — reason in one line]

## Structural gaps (required fields absent from the file)
| Required field | Status |
|---|---|
| [field] | Column not present in source |

## Gaps by field
| Field | Missing | % of in-scope | Severity | Indeterminate |
|---|---|---|---|---|
| [field] | [N] | [X]% | BLOCKER/DEGRADED/COSMETIC | [N] |

## Gaps by record
| Fields missing per record | Record count |
|---|---|
| 1 | [N] |
| 2 | [N] |
| 3+ | [N] |

## Record-level gap list (first 20 — full list: [filename].csv)
| Record ID | Source row | Field | Observed value | Severity |
|---|---|---|---|---|
| [id] | [n] | [field] | "" / " " / "N/A" | [severity] |

## Patterns observed
- [e.g. 184 of 212 phone gaps fall in rows 1,340–1,551, a contiguous block — likely export failure, verify before treating as real]

## Observed but out of scope
- [e.g. 47 email values present but malformed — not counted as missing]

## Next actions
| Field | Owner | Fix type | Blocking? |
|---|---|---|---|
| [field] | [name/team] | re-export / manual fill / schema change | yes/no |

## Assumptions
- [any decision you made without client confirmation]
```

## Failure modes

1. **Silent truncation.** The reader caps at 1,000 rows or stops at the first blank line, and you audit a fraction of the file while reporting a clean result. *Check:* compare the loaded row count to the file's own row count (line count, `COUNT(*)`, or the client's stated figure) and print both in the output header before auditing anything.
2. **Placeholder blindness.** A field is 100 percent populated but with `N/A`, `TBD`, or `.` and you report it as complete. *Check:* for every required field reporting zero or near-zero gaps, list its five most frequent values. If a non-meaningful token appears in the top five, add it to the blank definition and re-run.
3. **Wrong required-field list.** You audit against your own assumption instead of the client's spec, so the report flags fields nobody needs and misses one that blocks the import. *Check:* before running, paste the required-field list back to the client and get a yes. If you cannot, stamp PROVISIONAL on every page and list the fields you assumed.
4. **Column shift mistaken for missing data.** A malformed CSV shifts columns from a certain row onward, so a whole block reads as blank. *Check:* if gaps cluster in contiguous row ranges, open five raw rows from inside the block and five from outside and compare column alignment before reporting anything as missing.

## License

MIT
