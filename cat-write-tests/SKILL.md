---
name: cat-write-tests
description: >
  CAT tests explained: how to author them (add / edit / remove), the five expectations and
  which one answers which question, the test properties (key, sorting, tolerance, timeout,
  failure message). Obligatory reading whenever writing, changing or explaining a test, or
  choosing an expectation.
---

# Tests

A test is an entry under `Tests:` in the project file (`*.cat.yaml`): a query against a named
data source and an expectation about what comes back. You edit the file directly, so this
skill is not a sequence of steps: it is what you must read before you write, and what to check
after. Every fact is on the documentation page it links; the same page as markdown is at the
URL with `index.md` appended, which is the form to read when your fetch tool summarizes HTML.

## The shape

```yaml
Tests:
- Suite: Customers
  Name: No duplicate e-mails
  Description: Two customers with one e-mail break the login.
  Data source: DWH
  Query: |
    SELECT email FROM dbo.customers GROUP BY email HAVING COUNT(*) > 1
  Expectation: set is empty
```

`Name`, `Data source`, `Query` and `Expectation` are required; everything else is optional
and read only by the expectations that know it. Every property, its aliases, and the full
name that `Suite`, `Order`, `Test case` and `Name` form together:
https://docs.justcat.it/reference/tests/properties/

Keys and expectation names are matched loosely (`Data source`, `DataSource`, `data_source`
are one key; `set is empty` and `SetIsEmpty` are one expectation):
https://docs.justcat.it/reference/project-file/naming-conventions/

Tests may also live outside the project file: follow every `Get list of tests from:` entry,
and change a test where it is stored. https://docs.justcat.it/reference/project-file/lists/

## Pick the expectation by the question, never invent one

There are exactly five expectations. Pick by the question the test asks, then read the
expectation's page before writing the test:

- nothing should come back: `set is empty`
  https://docs.justcat.it/reference/tests/expectations/set-is-empty/
- something should be there: `set is not empty`
  https://docs.justcat.it/reference/tests/expectations/set-is-not-empty/
- exactly this many rows: `set row count`
  https://docs.justcat.it/reference/tests/expectations/set-row-count/
- these two sets are the same: `sets match`
  https://docs.justcat.it/reference/tests/expectations/sets-match/
- everything here is also there: `contains`
  https://docs.justcat.it/reference/tests/expectations/contains/

One page with an example of each and the question it answers, which is the page to read when
two of them seem to fit: https://docs.justcat.it/how-to-guides/test-patterns/which-expectation-when/

A condition that is none of the five (a count between two numbers, a sum within a percent of
last month, a ratio under a threshold, "the newest group is never higher than") is not a
missing expectation. The query selects the rows that break the rule, and the expectation is
`set is empty`. https://docs.justcat.it/how-to-guides/test-patterns/my-expectation-does-not-exist/

The query is written in the language of the data source's provider; the provider's page says
which. Load the data-sources skill when the project has no data source for the system the
test needs, or the query fails to connect.

## One set or two

`set is empty`, `set is not empty` and `set row count` read one set: `Data source` and
`Query`. `sets match` and `contains` compare two: `First data source` + `First query` and
`Second data source` + `Second query`, which may be two systems in two languages.
https://docs.justcat.it/reference/tests/expectations/introduction/

Before writing a two-set test, read these; the properties they describe are not optional
knowledge:

- Both sets must arrive sorted, `ORDER BY` in both queries or `Sort data: true` on the test;
  the same number of columns, compared by position, not by name; and a `Key` so the failure
  message can tell a different row from a missing one:
  https://docs.justcat.it/reference/tests/order-and-key/
- Numbers that may differ a little: `Tolerance` and `Tolerance mode`:
  https://docs.justcat.it/reference/tests/tolerance/
- How many offending rows the message shows and how long the values are, `Maximum errors
  logged` and `Maximum sample column length`, and what `Log number of errors` costs:
  https://docs.justcat.it/reference/tests/failure-message/

## Name it the way the project does

Read the existing `Tests:` first. Use the suites, test cases and tags the project already
has, and their naming style; a new suite or tag needs a reason. The test's `Description` says
why the rule matters, to a reader who sees only the failure. The four naming parts form the
full name that reports and filters use; CAT does not reject two tests with the same full
name, it reports both, so keep them distinct:
https://docs.justcat.it/reference/tests/properties/#full-name

## Check what you wrote

`catcli show -t` lists every test by full name, and fails when the project file does not
open; `-f <text>` narrows the list to one suite or test.
https://docs.justcat.it/reference/cat-cli/show/

Running the tests and reading the results is a decision for the user, not part of writing.
Tell the user which tests are new and what each will prove when run. What a result can be
(`Passed`, `Failed`, `Error`, `Inconclusive`) and what it carries:
https://docs.justcat.it/reference/tests/results/

## The traps

- `SELECT COUNT(*)` with `set is empty` always fails: one row always comes back. Select the
  offending rows instead.
- A property the expectation does not read is ignored without a warning: `Maximum errors
  logged` or `Log number of errors` on `set row count` does nothing; `Key`, `Sort data` and
  `Tolerance` do nothing on a one-set test. The "read by" column on the properties page says
  who reads what.
- `Second data source` or `Second query` on a one-set test is an error, with one exception:
  `set row count` also accepts its expected number in `Second query`, a form kept for older
  projects. Write `Expected row count`; when a test has both, CAT takes the first it finds
  and says nothing about the other.
- `set row count` reads the whole result to count it; keep the query small.
- `set is empty` stops after the first offending row unless `Log number of errors` is set,
  and then it reads everything. Leave it off unless something consumes the count.
- A two-set test with unsorted sets fails for the wrong reason, on rows that exist on both
  sides. A key with `NULL`s or duplicates is an error or a warning inside the message.
- `Timeout` is in seconds; `0` means no limit, and an expired test ends in `Error`.
- A plan may cap the number of tests in a project; the message names the cap. Do not work
  around it; report it.
