---
name: cat-propose-tests
description: >
  What is worth testing in a table, a database, a load or a pipeline, when the user has data
  and no tests, or too few: the order of value (the dumb "is there data" test before any
  rule, the rules before a comparison), the patterns most data tests follow, and how to look
  at the data first. Obligatory reading before proposing or choosing tests, before exploring
  the data on your own, whenever the user asks what to test or brings data without saying
  which tests they want.
---

# Propose tests

The user brings the data and a worry; the skill turns the worry into a few tests that earn
their run. Every fact is on the documentation page it links; the same page as markdown is at
the URL with `index.md` appended, which is the form to read when your fetch tool summarizes
HTML.

## Look before you propose

Read the project first: its data sources and the tests it already has, including the ones a
`Get list of tests from` entry points at. Existing tests are not a constraint, they are the
conventions: reuse their suites, tags and naming when a new test fits them, and only then. A
project with no tests is the common case and needs nothing more than the data sources.

Then look at the data through the project's own connection, so what you propose exists:

```
catcli exec -d <name> -c "<a statement in the provider's language>"
```

Through a pipe the rows arrive as CSV with a header row; an empty field is NULL and `""` an
empty string; the row count and a provider error go to standard error and the exit code
stays 0.

The tables and views, their columns and types, the keys, a `MIN` and `MAX` of the date
columns, a `COUNT` per obvious grouping, a `TOP 10`. Keep each statement small; a wide
`SELECT *` on a big table is the one thing not to run. https://docs.justcat.it/reference/cat-cli/exec/

Load the data-sources skill when the project has no data source for the system in question;
nothing can be proposed against data CAT cannot reach.

## Propose in value order

The order below is the order of value: each step only makes sense when the one before holds.
Propose a few tests at each step that applies, and say why each one matters, in the user's
words; do not propose every test that is possible.

1. **Is there data.** One `set is not empty` per table that matters, and per latest
   partition or period for anything loaded on a schedule; the run that catches the load that
   did not happen. Tag them `smoke`. Never `COUNT(*)`, which always returns a row.
   https://docs.justcat.it/how-to-guides/test-patterns/smoke-tests-with-set-is-not-empty/
2. **No row breaks the rule.** The rules the data must obey, one `set is empty` each with a
   query that selects the violators: keys unique, required columns not null, a foreign key
   with a parent, a value in its domain, a date not in the future, nothing older than the
   retention, a total that must never exceed another. The failure message hands over the
   culprits, which is why this is the shape of most data tests.
   https://docs.justcat.it/how-to-guides/test-patterns/find-problems-with-set-is-empty/
   Any condition that is not one of the five expectations is written this way:
   https://docs.justcat.it/how-to-guides/test-patterns/my-expectation-does-not-exist/
3. **Two systems agree.** When a source and a target, an old and a new, or a model and its
   warehouse hold the same data: climb the ladder, counts first, counts by period or key,
   then the IDs, then full rows only where the question needs it; each rung is one test with
   two queries. https://docs.justcat.it/how-to-guides/test-patterns/compare-data-across-systems/
   What the two systems will disagree on for no reason, and how to neutralize it in the
   query: https://docs.justcat.it/how-to-guides/test-patterns/differences-between-systems/
4. **The load keeps being right.** For an incremental load, counts per day over a window
   always, the rows on a short window, the full comparison once in a while:
   https://docs.justcat.it/how-to-guides/test-patterns/test-incremental-loads/
5. **The structure matches the design.** When there is a design, a spreadsheet or a
   catalog to compare against: tables exist, columns have the types they should:
   https://docs.justcat.it/how-to-guides/test-patterns/test-schemas-and-metadata/
6. **The rules with a deadline.** Masking, erasure, retention; the same `set is empty`
   shape, and the run is the evidence:
   https://docs.justcat.it/how-to-guides/test-patterns/test-compliance-rules/

Which of the five expectations answers which question, with an example of each:
https://docs.justcat.it/how-to-guides/test-patterns/which-expectation-when/

## From proposal to tests

Put the proposal in front of the user as a short list, each test as a sentence about the data
("no two customers share an e-mail") with the expectation and the cost in one clause, and let
them pick. Write the picked ones with the write-tests skill; when one test should become one
per table, partition or customer, the skill for generating tests from metadata does that with
a template. Running them is the user's decision.

## The traps

- A proposal made without looking at the data proposes tests on columns that do not exist.
  The `exec` step is not optional.
- Everything at once is nothing: a first suite is the smoke tests and the three or four
  rules the user would be embarrassed to have missed, not every rule that can be written.
- A comparison of two large tables as the first test fails loudly for the wrong reason when
  one side is empty; the smoke test comes first.
- Existing tests may already ask the user's question under another name; read them before
  proposing a duplicate.
- A rule the user states as a number ("between 1 and 3", "never higher than") is a query with
  the condition inverted and `set is empty`, not a missing expectation.
