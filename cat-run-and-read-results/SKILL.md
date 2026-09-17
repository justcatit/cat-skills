---
name: cat-run-and-read-results
description: >
  Running a CAT project and reading what happened: all tests or a part chosen by name, suite
  or tags, the console lines, the totals and the four results a test can end with. Use this
  skill whenever you are about to run tests, need to run only some of them, want to know how
  a run went, or need a pipeline or a script to decide on the results; running in parallel
  belongs here too.
---

# Run and read the results

A run opens the project, selects the tests that pass the filters, runs them, prints one line
per test and the totals, and writes the project's outputs. Every fact is on the documentation
page it links; the same page as markdown is at the URL with `index.md` appended, which is the
form to read when your fetch tool summarizes HTML.

## Run all of it, or a part

```
catcli run
catcli run -f "Customers.*"
catcli run -i "smoke, nightly" -e ManualOnly
catcli run -n -l Information
```

`-f` is a pattern matched against the whole full name (`[Suite].[Order].[Test case].[Name]`):
`*` any text, `?` one character, not case-sensitive, brackets ignored — `sales.*` selects a
suite, `*orders*` every test with orders in its name. `-i` and `-e` select by tags; the three
combine. `-n` skips every output for an ad-hoc run. `-l Error` prints the failed tests'
messages as they finish; the default prints only the result lines. Every option, and the
examples: https://docs.justcat.it/reference/cat-cli/run/

To see what a filter would select before running it: `catcli show -t -f "<pattern>"`.
https://docs.justcat.it/reference/cat-cli/show/

The same run from CAT Studio, the PowerShell module and the Python module:
https://docs.justcat.it/how-to-guides/organize-and-run-tests/run-your-tests/

## Read what came back

Each test ends with exactly one of four results: `Passed`, `Failed` (the expectation does not
hold; the message says why), `Error` (the expectation could not be evaluated: a missing data
source, a statement the provider refused, a timeout, a wrong property) or `Inconclusive`
(not run, because the plan caps the tests per run). A test filtered out has no result at all.
What each means and what a result carries:
https://docs.justcat.it/reference/tests/results/

Failed is a statement about the data; Error is a statement about the test or the connection.
Treat them differently, and load the skill for a failed test before explaining either.

The console shows a line per test and the totals. The details of a failed test, the message
with the sample of offending rows and the queries, are in the project's outputs, or on the
console with `-l Error`. Load the outputs skill when the results must be kept or read by a
program; without an output only the console text exists.

To step through the failed tests one by one without reloading the project, and to run again
after a fix: `catcli open`, then `run`, `result`, `rerun`.
https://docs.justcat.it/reference/cat-cli/open/

## The exit code

`run` exits `0` when every selected test passed; `10` when any did not (`Failed`, `Error` or
`Inconclusive`), or when the filter selected nothing. `--exitCode run` restores `0` whenever
the run finished, whatever the results. `1` means the project did not open or the run aborted. `2` to `8` are sign-in and plan
problems, not test problems; report what the message says and do not work around them:
https://docs.justcat.it/reference/cat-cli/introduction/#exit-codes
https://docs.justcat.it/reference/cat-cli/run/#exit-code

A pipeline or a script that needs the picture rather than the verdict still reads an output:
`junit` or `trx` for a test report the platform shows, `json` for a script; the totals block
on the console is the fallback.

## Several tests at once

`Threads` at the root of the project file runs that many tests in parallel, default one. It
multiplies the load on the systems under test; when it helps and when to leave it alone:
https://docs.justcat.it/how-to-guides/organize-and-run-tests/run-tests-in-parallel/
https://docs.justcat.it/reference/project-file/root-settings/

## The traps

- Exit code `10` with zero tests in the totals means the filter matched nothing, not that
  tests failed.
- Exit code `10` says nothing about which tests failed or why; read the totals and the
  output for that.
- `Inconclusive` is not a failure and not a pass: the plan's cap on tests per run left the
  test unrun. The count is in the totals; the fix is on the plan side, not in the project.
- A bare word is a whole name: `-f orders` selects nothing, `-f "*orders*"` selects by a
  part. Quote a filter with spaces.
- A run with `-n` writes no output; a second run without `-n` is needed for the file or the
  table.
- The project file is read fresh on every run; `open`'s `run` does not re-read it, `rerun`
  does.
