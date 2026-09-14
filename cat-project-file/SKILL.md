---
name: cat-project-file
description: >
  The CAT project file (*.cat.yaml) is the single input every CAT tool opens: which data to
  connect to, what to test, where the results go, all in one file or pointed to from it.
  Without it there is nothing to run. Use this skill whenever you create a project, read or
  restructure a project file, move definitions into other files or a database, or a tool
  says it cannot open the project; the content of data sources, tests and outputs each have
  a skill of their own.
---

# The project file

A project is one `<name>.cat.yaml` file. Every CAT tool opens it, resolves the definitions
it holds or points to, runs the tests and writes the outputs. You edit the file directly, so
this skill is not a sequence of steps: it is what you must read before you write, and what to
check after. Every fact is on the documentation page it links; the same page as markdown is
at the URL with `index.md` appended, which is the form to read when your fetch tool
summarizes HTML.

## The shape

```yaml
Data sources:
- Name: DWH
  Provider: SqlServer@2
  Connection string: "%DWH_CONNECTION_STRING%"
Tests:
- Name: Departures are loaded
  Data source: DWH
  Query: SELECT TOP 1 * FROM FACT.DEPARTURES
  Expectation: set is not empty
Output: xlsx
Threads: 1
```

Every key sits at the root of the document with no indentation. `Data sources` and `Tests`
are required, in the file or pointed to from it; `Queries`, `Output` and `Threads` are
optional. What each key holds, how a tool finds the file (a path, a folder with exactly one
`*.cat.yaml`, or the working directory), and the file name rule:
https://docs.justcat.it/reference/project-file/introduction/

One file that exercises everything the project file accepts, with a walk-through; read it
before restructuring anything: https://docs.justcat.it/reference/project-file/complete-example/

## Each section has its skill

This skill owns the file; the content of the sections is owned elsewhere. Load the
data-sources skill before writing under `Data sources`, the write-tests skill before writing
under `Tests`, the outputs skill before writing under `Output`. `Queries` holds named
statements with two uses only, test templates and saved queries in CAT Studio; a test's own
`Query` is statement text, never a query name:
https://docs.justcat.it/reference/queries/properties/

`Threads` is the one root setting: how many tests run at once, default one:
https://docs.justcat.it/reference/project-file/root-settings/

## No file yet

`catcli new` creates a project from a template, into the current folder or a wrapped
sub-folder; `--list` shows the templates. The other tools create one too, and CAT Studio
writes one when it saves:
https://docs.justcat.it/reference/cat-cli/new/
https://docs.justcat.it/how-to-guides/organize-and-run-tests/how-to-create-project-file/

## Definitions kept elsewhere

CAT reads every definition from a list, and the project file is only one place a list can
point at. `Get list of data sources from`, `Get list of queries from` and `Get list of tests
from` each name a provider, a connection string and a query that returns one definition per
row: another YAML file, a worksheet, a database table, a stored procedure. Both flavours mix
in one file. Read the whole file and follow every such entry before saying what a project
contains, and change a definition where it is stored, not in the project file:
https://docs.justcat.it/reference/project-file/lists/

When one file stops being enough, how to split it and where definitions can live, and the
scripts for keeping tests in database tables:
https://docs.justcat.it/how-to-guides/organize-and-run-tests/organize-test-definitions/
https://docs.justcat.it/how-to-guides/organize-and-run-tests/store-definitions-in-a-database/

## The dialect

Keys and property names are matched loosely: `Data sources`, `Data_Sources` and
`DataSources` are one key; casing is free; several properties answer to more than one name.
Keep one style within one file, and know that queries and connection strings are handed to
the provider exactly as written:
https://docs.justcat.it/reference/project-file/naming-conventions/

`%NAME%` in any value, in any file or table a list points at, is replaced with the
environment variable `NAME` when it exists; keys are never expanded. This is how secrets stay
out of the file and how one file runs against several environments:
https://docs.justcat.it/reference/project-file/environment-variables/

## Check what you wrote

`catcli show -s` opens the project and prints what it resolved: every list, every data
source with the number of tests using it, the suites, the settings. It exits `1` with the
reason when the project does not open; `-l Information` shows where loading stopped. `-t`
lists the tests by full name. https://docs.justcat.it/reference/cat-cli/show/

Any exit code from 2 to 7 is a sign-in or plan problem, not a file problem. Do not work
around it; report what the message says.
https://docs.justcat.it/reference/cat-cli/introduction/#exit-codes

## The traps

- Two `*.cat.yaml` files in one folder, or none, and a tool given the folder refuses; pass
  the file's path. A file without the `.cat.yaml` extension is not found by folder lookup.
- A root key written with indentation, or under another key, is silently a property of
  that definition, not a section.
- A multi-line query needs a YAML block (`Query: |`); a colon followed by a space inside a
  plain scalar breaks the YAML. Quote a connection string that holds `#` or starts with
  `%`.
- A `LIKE '%PATH%'` or `'%TEMP%'` in a query is expanded, because those variables exist on
  every machine; the page shows how to keep the percent signs apart from the name.
- `Named sets` is the old name of `Queries` and `Outputs` of `Output`; both are still read.
  Write the current names.
- A project with `Tests:` but no data source, or the reverse, does not open, and the message
  names the missing key.
