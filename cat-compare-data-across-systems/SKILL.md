---
name: cat-compare-data-across-systems
description: >
  Two systems should hold the same data (a source and a warehouse, a lakehouse and a Power
  BI model, an old and a new system, a table and its copy) and the test must say whether
  they do and where they differ: one test with two queries, climbed as a ladder from counts
  to full rows, with the rules that make two systems comparable. Obligatory reading before
  writing any test that compares two result sets, before choosing between `sets match` and
  `contains`, and when such a comparison fails on rows that look identical.
---

# Compare data across systems

One test, two queries, two data sources in two technologies if need be; CAT steps through
both result sets side by side and reports the rows that differ. Every fact is on the
documentation page it links; the same page as markdown is at the URL with `index.md`
appended, which is the form to read when your fetch tool summarizes HTML.

## Climb the ladder, in order

Counts, then counts by period or key, then the IDs, then the full rows; each rung costs
more and proves more, and a failing count makes the rungs above pointless. Most projects
keep tests on every rung: the cheap ones after every load, the expensive ones nightly or on
demand. The four rungs with a test each, and the tricks for big volumes (checksums per
group, IDs everywhere and full rows on a window, sampling by key, hashing within one
technology only): https://docs.justcat.it/how-to-guides/test-patterns/compare-data-across-systems/

`sets match` when both sides must be the same; `contains` when one side may hold more (a
permanent staging area, a history table), the superset as the first query or written as
`second contains first`:
https://docs.justcat.it/reference/tests/expectations/sets-match/
https://docs.justcat.it/reference/tests/expectations/contains/

## The rules that make two systems comparable

- Both sets ordered the same way, `ORDER BY` the key on both sides, or `Sort data: true`
  for small sets; a key on a number, a date or an ID, never text; the same number of
  columns, compared by position: https://docs.justcat.it/reference/tests/order-and-key/
- Values compare leniently on purpose (NULL equals NULL, text ignoring case, numbers
  within `Tolerance`, dates as dates); what still bites, numeric types, dates against
  date-times, time zones, NULL against empty string, trailing spaces, booleans, GUIDs,
  text sort orders, and the fix for each, on both sides in the query:
  https://docs.justcat.it/how-to-guides/test-patterns/differences-between-systems/
  https://docs.justcat.it/reference/tests/tolerance/
- `Maximum errors logged` decides how far CAT reads after the first difference; the
  message says whether the scan was complete: https://docs.justcat.it/reference/tests/failure-message/

Write the test with the write-tests skill; a smoke test on each side comes first, since a
comparison against an empty table fails loudly for the wrong reason.

## The traps

- Rows present on both sides reported as missing, or a warning about a decreasing key:
  the two sides are sorted by different rules; sort by a numeric or date key.
- A hash of the row across two different technologies never matches; hash within one
  technology, select the columns across two.
- A source written continuously differs in its newest minutes on every run; cut both
  sides at the same moment in the past, in UTC.
- Rung 4 on the whole table is the migration test run once, not the nightly test; the
  nightly test is rung 4 on a window and rung 2 on everything.
