---
name: cat-find-problems
description: >
  A rule the data must obey, written as the query that finds the rows breaking it and the
  expectation that nothing comes back: orphans, duplicates, nulls where none may be, values
  out of range, a date in the future, a total over a threshold, anything "no row may". The
  shape of most data tests. Obligatory reading before writing any `set is empty` test, and
  whenever a rule sounds like a missing expectation (a count between two numbers, a sum
  within a percent, a gap in a sequence).
---

# Find problems with set is empty

State the rule as a violation query: every row it returns is a problem, and the failure
message hands over the culprits. Every fact is on the documentation page it links; the same
page as markdown is at the URL with `index.md` appended, which is the form to read when your
fetch tool summarizes HTML.

## The pattern

Select the columns that identify the row and show the problem, the key and the offending
value, not `*`; that is what will be read in the message. The examples, what comes back when
it fails, the three properties that size the message, and the cost trick (`TOP 1` or `LIMIT 1`
when the count is not needed): https://docs.justcat.it/how-to-guides/test-patterns/find-problems-with-set-is-empty/

The expectation's page, with `Maximum errors logged`, `Log number of errors` and the
`COUNT(*)` trap: https://docs.justcat.it/reference/tests/expectations/set-is-empty/

## A rule that sounds like a missing expectation

A count between two numbers, a sum within a percent of last month, a ratio under a
threshold, no gap in a sequence, the newest group never higher than a bound: none is an
expectation, every one is a query that selects the violation, and `set is empty`:
https://docs.justcat.it/how-to-guides/test-patterns/my-expectation-does-not-exist/

## The traps

- `SELECT COUNT(*)` with `set is empty` always fails: one row always comes back. Select the
  offending rows.
- The default sample is one row and the scan stops there; raise `Maximum errors logged` to
  see a pattern, set it to `0` when the rows are personal data, and leave `Log number of
  errors` off unless something consumes the total.
- A rule with two systems in it (every order's customer exists in the CRM) is a two-set
  test; load the compare skill.
- A violation query with a `LIKE '%PATH%'` in it is rewritten by environment-variable
  expansion; keep the percent signs apart from the name.
