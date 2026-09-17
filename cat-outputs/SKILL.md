---
name: cat-outputs
description: >
  Outputs are needed for preserving the history of test results. Without outputs, only the
  standard output and error are used, nothing else. Whenever it might be useful to save the
  results of tests to a JSON, MS Excel or other file, or to a database table, use this skill
  to configure the necessary outputs.
---

# Outputs

An output is where the results of a run end up. It is requested under the root key `Output`
of the project file (`*.cat.yaml`), at the start of a line like `Threads`, never inside a test
or a data source. You edit the file directly, so this skill is not a sequence of steps: it is
what you must read before you write, and what to check after. Every fact is on the
documentation page it links; the same page as markdown is at the URL with `index.md`
appended, which is the form to read when your fetch tool summarizes HTML.

## The shape

```yaml
Output: xlsx, json
```

```yaml
Output:
- Format: junit
  File: TestResults/cat-junit.xml
- Database: DWH
  Table: dbo.CatTestResult
```

The short form is format names only, each a file with the default name; the list form is one
item per output, a file (`Format`, `File`) or a database (`Database` with `Table` or
`Procedure`), the two kinds mixed in one list. Every key, the default file name, relative
paths, and what `{timestamp}` does: https://docs.justcat.it/reference/outputs/settings/

## Pick the format by who reads it

Read the table of formats first, then the page of the one you pick; every format page shows
the same sample run, so the pages are comparable:
https://docs.justcat.it/reference/outputs/overview/

- A program, a pipeline step that decides on the results, a script: `json`
  https://docs.justcat.it/reference/outputs/json/
- The same data for reading by eye: `yaml` https://docs.justcat.it/reference/outputs/yaml/
- A person, a file to hand over; the one format every plan writes: `xlsx`
  https://docs.justcat.it/reference/outputs/xlsx/
- A test report in Azure DevOps, GitLab, GitHub Actions or Jenkins: `junit`
  https://docs.justcat.it/reference/outputs/junit/
- A test report in Azure DevOps' publish task or Visual Studio: `trx`
  https://docs.justcat.it/reference/outputs/trx/
- History across runs, a dashboard, something that must happen the moment a test fails: a
  database output, one row per test as it finishes, into a table or through a procedure:
  https://docs.justcat.it/reference/outputs/database-outputs/

What every output carries for each test, the result and the whole definition, and the
column or parameter names CAT recognizes: https://docs.justcat.it/reference/outputs/properties/

## A database output is a data source plus a target

`Database` names a data source the project already defines; its provider must be one of the
four the page lists. Load the data-sources skill when the project has no data source for the
database that should receive the results. Then read the database's own page for the script
that creates the table or the procedure, and create the objects from it rather than letting
CAT create them with a privileged account:

- SQL Server https://docs.justcat.it/reference/outputs/sqlserver/
- PostgreSQL https://docs.justcat.it/reference/outputs/postgres/
- Oracle https://docs.justcat.it/reference/outputs/oracle/

## Check what you wrote

The check is a run that writes the outputs, because that is when every output is
validated: a wrong database item fails the open, a wrong format name fails the start of
the run, before any test, and a file that cannot be written fails at the end. Run one small
test with a name filter rather than the whole project:

```
catcli run -f "*<part of one test's full name>*"
```

`-n` skips every output, so it does not check them; `show` opens the project but does not
look at the format names. Then look at the file, or at the table. After a run, `run` exits
`0` whatever the results, so a pipeline that must fail on a failed test reads an output,
`junit` or `json`, not the exit code. https://docs.justcat.it/reference/cat-cli/run/

Any exit code from 2 to 8 is a sign-in or plan problem. The Starter and Professional plans
write `xlsx` only; every other format and every database output is refused on them, and the
message says what the Team plan adds. Do not work around it; report what the message says.
https://docs.justcat.it/reference/cat-cli/introduction/#exit-codes

## The traps

- A format name that is not one of `xlsx`, `json`, `yaml`, `trx`, `junit` (such as `excel`, `csv`,
  `xml`) ends the run with `Unknown format of output file` before any test runs.
- `Output` inside a test or a data source is silently a property of that test or data source,
  not an output. It is a root key.
- A database item needs exactly one of `Table` and `Procedure`; neither or both fails the
  open. A data source with another provider than the four fails the open.
- Without `{timestamp}` in `File`, every run overwrites the file; with it, every run adds
  one. A pipeline wants the fixed name, a history wants the timestamp or a database.
- A relative `File` is resolved against the project file's directory, not the current
  directory; a missing directory is created.
- Columns of a database table are matched by name, not by position; a column CAT does not
  recognize must be nullable or have a default, or every insert fails.
- `Path` is the older name of `File`; write `File`.
