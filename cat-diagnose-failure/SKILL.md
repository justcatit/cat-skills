---
name: cat-diagnose-failure
description: >
  A CAT test did not pass, and the message has to be read right before anything is changed:
  Failed means the data broke the rule, Error means the test or the connection is broken,
  Inconclusive means the plan left it unrun, and each message has a fixed shape per
  expectation. Obligatory reading when a test fails or errors and the user or the agent needs
  to understand why, before changing the test, the data or the data source.
---

# Diagnose a test that did not pass

The result tells you what kind of problem it is, and the message tells you where. Read both
before touching anything; a test that fails because the data is wrong is doing its job, and
"fixing" it hides the finding. Every fact is on the documentation page it links; the same
page as markdown is at the URL with `index.md` appended, which is the form to read when your
fetch tool summarizes HTML.

## Get the message

The console line shows only the result. The message is in the project's outputs, or on the
console when the run uses `-l Error`, or one test at a time with `catcli open` and its
`result` command, which shows the description, the message and both queries side by side.
Re-run one test by name rather than the project: `catcli run -f "*.<name>" -n -l Error`.
https://docs.justcat.it/reference/cat-cli/run/
https://docs.justcat.it/reference/cat-cli/open/

## First the result, then the message

- **Failed**: the queries ran and the expectation does not hold. This is a finding about the
  data, unless the test itself asks the wrong question. Read the message against the
  expectation's shape below.
- **Error**: the expectation could not be evaluated. The message names the cause: a data
  source or query the project does not define, a statement the provider refused (with the
  provider's own error text), a second-set property on a one-set expectation, a key that
  cannot be compared, a `NULL` in a key, an expired `Timeout`.
- **Inconclusive**: the test was not run, because the plan caps the number of tests per run.
  Nothing in the project is wrong. Report it; do not work around it.

What each result means and what a result carries:
https://docs.justcat.it/reference/tests/results/

## Read a Failed message by its expectation

The message has a fixed shape per expectation. Read the expectation's page for the shape and
the properties that size the message:

- `set is not empty`: no row came back. The table or the filter is empty, or the query
  selects the wrong thing. https://docs.justcat.it/reference/tests/expectations/set-is-not-empty/
- `set is empty`: at least one row came back, and a sample of them is in the message; "CAT
  did NOT scan the entire set" means there may be more. Those rows are the finding. If the
  query is `SELECT COUNT(*)`, the test is wrong, not the data: one row always comes back.
  https://docs.justcat.it/reference/tests/expectations/set-is-empty/
- `set row count`: the expected and the actual count are both in the message.
  https://docs.justcat.it/reference/tests/expectations/set-row-count/
- `sets match` and `contains`: a table of differing rows, `<` and `>` for the two sides, `!`
  on the values that differ, and whether the scan was complete; with a key, missing rows and
  different rows are told apart. A difference on rows that exist on both sides, or a warning
  about a decreasing or duplicate key, usually means the sets are not sorted the same way,
  which is a test problem, not a data problem. How to read the table, and how `Maximum
  errors logged` changes it: https://docs.justcat.it/reference/tests/failure-message/
  https://docs.justcat.it/reference/tests/order-and-key/

## Decide what to change

- The data is wrong and the test is right: report the finding with the sample; change
  nothing in the project.
- The test asks the wrong question (the wrong expectation, an unsorted two-set query, a
  count with `set is empty`, a wrong column order): load the write-tests skill and change the
  test.
- The connection is wrong (login, server, driver, a `%NAME%` left unreplaced): the message
  carries the provider's error; load the data-sources skill and check the connection there.
- The statement is wrong (an unknown object, a syntax error): the data source is fine; fix
  the query in the test.
- A `Timeout` expired: the query is too slow for the limit; the test's `Timeout` and the
  query itself are the two places to look.
  https://docs.justcat.it/reference/tests/properties/

Then run that one test again with the same command, and read the result again.

## The traps

- Exit code `0` does not mean the tests passed, and `1` does not mean a test failed: `1` is
  a project that did not open. Codes `2` to `8` are sign-in and plan problems.
  https://docs.justcat.it/reference/cat-cli/introduction/#exit-codes
- With `Maximum errors logged` at its default of `1`, the message shows one offending row and
  says the scan was incomplete; raising it to see more costs a full read on a large set.
- A `sets match` message counts a differing row as two differences without a key and as one
  with a key.
- A test filtered out of the run has no result; a test that is not in the totals was not
  selected, not passed.
