---
name: cat-generate-tests-from-metadata
description: >
  One working CAT test that should become one test per table, partition, customer,
  contract or any family a query can list: a template with a `Metadata` query, expanded
  into one test per row when the project opens, with `%COLUMN%` placeholders in every
  property. Obligatory reading before writing a test with a `Metadata` property, before
  copying a test many times with small changes, and when a project's test count is not
  what its file suggests.
---

# Generate tests from metadata

A test whose `Metadata` property names a query definition is a template: CAT runs that
query when the project opens and produces one test per returned row, replacing `%COLUMN%`
in every property with the row's value. The template itself is not run. Every fact is on the
documentation page it links; the same page as markdown is at the URL with `index.md`
appended, which is the form to read when your fetch tool summarizes HTML.

## The shape

```yaml
Queries:
- Name: staging tables
  Data source: DWH
  Query: SELECT TABLE_SCHEMA, TABLE_NAME FROM INFORMATION_SCHEMA.TABLES WHERE TABLE_SCHEMA = 'stage'
Tests:
- Name: Staging table %TABLE_SCHEMA%.%TABLE_NAME% is not empty
  Metadata: staging tables
  Tags: smoke
  Data source: DWH
  Query: SELECT TOP 1 * FROM [%TABLE_SCHEMA%].[%TABLE_NAME%]
  Expectation: set is not empty
```

What a template is, the substitution rules, what happens with no rows, with a bad query
name, and with the plan's cap: https://docs.justcat.it/reference/tests/templates/

The metadata query is an ordinary query definition; its properties:
https://docs.justcat.it/reference/queries/properties/

## The three steps

Define the family as a query, write the test for one member and make sure it passes, then
replace the specific values with placeholders and add `Metadata`. The how-to, with when it
pays off and the tips: https://docs.justcat.it/how-to-guides/organize-and-run-tests/generate-tests-from-metadata/

Write the one-member test with the write-tests skill first; a template made from a test
that never ran generates the same mistake once per row.

## Check what you wrote

`catcli show -t` lists the generated tests by full name, one per row, and the template is
not among them; `catcli show -s` has the count. A template whose query names a missing
query or data source, or whose query fails, fails the open, and the message names the
template. https://docs.justcat.it/reference/cat-cli/show/

## The traps

- Placeholders are column names as the query returned them, case included: `%TABLE_NAME%`
  and `%table_name%` are different placeholders, and a placeholder no column matches stays
  as text.
- Environment variables are replaced first, then columns: a column named like a variable
  that exists on the machine loses.
- A `NULL` in a row becomes an empty string, silently.
- A query returning no rows produces no tests and no error; a project that "lost" tests
  usually has a metadata query that now returns nothing.
- Every generated test counts towards the plan's tests per project; a family of a thousand
  tables is a thousand tests.
- `Name` and `Expectation` are required on the template as on a test, and the name must
  carry a placeholder, or every generated test has the same full name.
