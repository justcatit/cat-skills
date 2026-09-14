---
name: cat-smoke-tests
description: >
  There should be data where data should be: the dumb test before any clever one, one row
  is enough, per table, per latest partition or period, per business slice, and generated
  for every table so the new one gets its test too. Obligatory reading before writing a
  "not empty" test, before deciding what runs first after a load, and when a run of
  comparisons fails everywhere at once.
---

# Smoke tests

Is there data? One row is enough, the test costs almost nothing, and when it fails nothing
else is worth running. Every fact is on the documentation page it links; the same page as
markdown is at the URL with `index.md` appended, which is the form to read when your fetch
tool summarizes HTML.

## The pattern

Three grains: the table, the latest partition or period, a business slice. The middle one
catches the load that did not run; it runs first after every load and before the expensive
comparisons. The tests, the grains, and the pairing with a freshness check when "not empty"
is not enough: https://docs.justcat.it/how-to-guides/test-patterns/smoke-tests-with-set-is-not-empty/

The expectation and the one trap it has: https://docs.justcat.it/reference/tests/expectations/set-is-not-empty/

## Every table, without writing every test

One template over the catalog (the tables of a schema, the partitions of a control table,
the customers), tagged `smoke`, and the table created next week gets its test on the next
open; load the skill for generating tests from metadata:
https://docs.justcat.it/reference/tests/templates/

Run the smoke suite alone with the tag filter before anything else:
https://docs.justcat.it/how-to-guides/organize-and-run-tests/run-your-tests/

## The traps

- `SELECT COUNT(*)` with `set is not empty` always passes: one row always comes back.
  Select a row (`TOP 1`, `LIMIT 1`), not a count.
- A table with one stale row passes; pair the smoke test with a freshness rule (`set is
  empty` on the newest timestamp being too old) or an exact `set row count` where the count
  is known.
- A smoke test per table that does not use a template is a test the new table never gets.
