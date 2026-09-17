---
name: cat-organize-suite
description: >
  A CAT project that has outgrown one file: how to split tests into several YAML files,
  keep them in database tables or a workbook the business maintains, name suites, test
  cases and tags so that runs can be filtered, and what CAT Studio does with definitions it
  did not write. Use this skill whenever a project has too many tests for one file, tests
  should be maintained by people who do not write YAML, or the suite needs a structure that
  runs can select by.
---

# Organize a growing suite

Every definition has the same properties wherever it is stored; the project file either
holds it or points at where it is. Every fact is on the documentation page it links; the
same page as markdown is at the URL with `index.md` appended, which is the form to read when
your fetch tool summarizes HTML.

## When one file stops being enough, and the three shapes

Too many tests for one file, tests that are about a database anyway, or people who know
the rules but not YAML: the page has the three shapes, several YAML files, a database, a
workbook, and how they mix with inline definitions:
https://docs.justcat.it/how-to-guides/organize-and-run-tests/organize-test-definitions/

The mechanism behind all three, `Get list of ... from` with a provider, a connection string
and a query returning one definition per row, and how lists combine:
https://docs.justcat.it/reference/project-file/lists/

For a database: create only the tables you need, the columns, the scripts per platform:
https://docs.justcat.it/how-to-guides/organize-and-run-tests/store-definitions-in-a-database/

Load the project-file skill for the file itself and the data-sources skill when the list
lives in a database that needs a data source.

## Structure that runs can select by

`Suite`, `Order`, `Test case` and `Name` form the full name; `Tags` are free labels. A run
selects by a pattern on the full name (`*` any text) or by tags, so the structure is what the
team will want to run separately: a `smoke` tag for the fast checks, a suite per area or per source
system, tags per schedule (`nightly`, `hourly`) or per environment. Names need not be
unique, but two tests with one full name are reported twice, ambiguously:
https://docs.justcat.it/reference/tests/properties/#full-name
https://docs.justcat.it/how-to-guides/organize-and-run-tests/run-your-tests/

Property names may follow each storage's style (`test_suite` in a database, `Test Suite`
in a worksheet, `TestSuite` in YAML); within one document keep one style:
https://docs.justcat.it/reference/project-file/naming-conventions/

One test that should exist per table, customer or contract is a template, not a copy;
load the skill for generating tests from metadata.

## Moving definitions without breaking the project

- Move tests verbatim; the properties and their names do not change with the storage.
- Compare `catcli show -s` and `catcli show -t` before and after: the same suites, the same
  count, the same data sources in use. https://docs.justcat.it/reference/cat-cli/show/
- Paths in a list entry are relative to the project file; `%NAME%` works in them.
- CAT Studio reads every list and shows the definitions; it saves only what it can write
  back. What it does with definitions from a database or a worksheet is on the organize
  page's last section.

## The traps

- A list entry's `Query` for a YAML file is a node path (`/Tests`), for a database a
  statement, for a workbook a worksheet select; the provider's page says which.
- A database list needs a data source's worth of connection, with the secret in `%NAME%`,
  not in the entry.
- Column names in a table or a header row are matched by the same synonyms as YAML keys;
  a column no property answers to is ignored without a warning.
- A suite named in the project file and the same suite in a separate file are one suite;
  nothing prevents it, and nothing warns when a test lands in both.
