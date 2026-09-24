---
name: cat-data-sources
description: >
  CAT data sources explained: how to author them (add / edit / remove), description of what
  technology is behind, how to work with secrets. Obligatory reading whenever authoring data
  sources or troubleshooting tests or queries that fail because of connectivity.
---

# Data sources

A data source is a named entry under `Data sources:` in the project file (`*.cat.yaml`); tests
refer to it by that name. You edit the file directly, so this skill is not a sequence of
steps: it is what you must read before you write, and what to check after. Every fact is on
the documentation page it links; the same page as markdown is at the URL with `index.md`
appended, which is the form to read when your fetch tool summarizes HTML.

## The shape

```yaml
Data sources:
- Name: DWH
  Provider: SqlServer@2
  Connection string: "%DWH_CONNECTION_STRING%"
```

`Name`, `Provider` and `Connection string` are the properties every provider reads; anything
else on the entry is a setting of the provider. What each property means, the aliases the
file accepts (`File path`, `Folder`), and the optional `Technology` line that only CAT Studio
reads: https://docs.justcat.it/reference/data-sources/properties/

No project file yet: `catcli new` writes one from a template.
https://docs.justcat.it/how-to-guides/organize-and-run-tests/how-to-create-project-file/

## Find your provider, then read its page

Never write a connection string from memory. Find the system in the provider catalog, then
read that provider's page (list at the end) before writing anything: it has the connection
string keys, the authentication modes with an example each, the settings the provider accepts,
what must be installed, and the query language.

- The catalog, one row per provider, with what it connects to and platform notes; one
  provider serves many systems, and anything with an ODBC driver goes through `Odbc@1`:
  https://docs.justcat.it/reference/data-sources/providers/overview/
- The full list of systems CAT can test with the provider behind each, and the `Technology`
  code to add when the file will also be opened in CAT Studio:
  https://docs.justcat.it/reference/data-sources/technologies/

## Secrets

A password, a token or a whole connection string never goes into the file. Write `%NAME%`
where the value belongs and put the value in the environment variable `NAME`; CAT replaces it
when it opens the project. The rule and the alternatives, integrated authentication and
Microsoft Entra ID: https://docs.justcat.it/how-to-guides/organize-and-run-tests/work-with-secrets/

You can set the variable in your own shell to run the check below, but that shell is not the
user's session and not the pipeline. Tell the user which variables the project needs and
where to define them so every run sees them; the how-to has the ways and the check:
https://docs.justcat.it/how-to-guides/organize-and-run-tests/use-environment-variables/

## Check the connection, read the result

Run one trivial statement in the provider's language through CAT, with the project's own
connection and environment:

```
catcli exec -d <name> -c "SELECT 1"
```

Through a pipe the rows arrive as CSV with a header row; an empty field is NULL and `""` an
empty string; the row count and a provider error go to standard error and the exit code
stays 0.

The provider's error comes back in the output with exit code 0; exit code 1 means the project
did not open or the name is wrong; `-l Information` shows what CAT does while it opens the
project. https://docs.justcat.it/reference/cat-cli/exec/

Any exit code from 2 to 8 is a sign-in or plan problem, not a data source problem. Do not
work around it; report what the message says. The codes:
https://docs.justcat.it/reference/cat-cli/introduction/#exit-codes

When the statement fails, read the error against these, in order:

- `%NAME%` still in the value, or an invalid connection string: the variable is not visible
  to the process running `catcli`; the how-to above has the check.
- Login failed, authentication, permission: the credentials or the authentication mode, on
  the provider's page.
- Server not found, network, timeout, certificate: server name, port, firewall, VPN, and the
  certificate keys the provider's page shows for a local or self-signed server.
- A driver or a component missing: the prerequisites on the provider's page.
- An unknown object, a syntax error: the statement, not the connection; the data source is
  fine and the problem belongs to the test or the query.

## Before you add, change or remove

- Read the whole `Data sources:` list first, and follow every `Get list of data sources
  from:` entry: data sources may also be served from another file or a database, and those
  are changed where they are stored, not in the project file.
  https://docs.justcat.it/reference/project-file/lists/
- A data source that already points at the same system is the one to use. CAT does not
  detect two entries for one system, and a test would use either.
- Renaming: tests refer to the data source by `Data source`, `First data source` and `Second
  data source`, queries by `Data source`; the keys are also accepted with underscores or
  without spaces and in any case (`DATA_Source`, `FirstDataSource`), and the match on the
  name ignores case. Change every reference, in the project file and in every file a list
  points at, then run the project: a test still on the old name ends in an error that names
  it and lists the data sources that do exist.
  https://docs.justcat.it/reference/project-file/naming-conventions/
- Switching provider: the connection string starts over on the new provider's page; the old
  one does not carry over.
- Removing: delete the entry and every test that uses it, or point those tests elsewhere.

## Provider pages

ClickHouse@1 https://docs.justcat.it/reference/data-sources/providers/clickhouse-1/ ·
Csv@2 https://docs.justcat.it/reference/data-sources/providers/csv-2/ ·
Databricks@1 https://docs.justcat.it/reference/data-sources/providers/databricks-1/ ·
Dax@2 https://docs.justcat.it/reference/data-sources/providers/dax-2/ ·
Excel@2 https://docs.justcat.it/reference/data-sources/providers/excel-2/ ·
MySql@1 https://docs.justcat.it/reference/data-sources/providers/mysql-1/ ·
Odbc@1 https://docs.justcat.it/reference/data-sources/providers/odbc-1/ ·
Oracle@1 https://docs.justcat.it/reference/data-sources/providers/oracle-1/ ·
Postgres@1 https://docs.justcat.it/reference/data-sources/providers/postgres-1/ ·
PowerBI@2 https://docs.justcat.it/reference/data-sources/providers/powerbi-2/ ·
Snowflake@1 https://docs.justcat.it/reference/data-sources/providers/snowflake-1/ ·
SqlServer@2 https://docs.justcat.it/reference/data-sources/providers/sqlserver-2/ ·
Teradata@1 https://docs.justcat.it/reference/data-sources/providers/teradata-1/ ·
Yaml@1 https://docs.justcat.it/reference/data-sources/providers/yaml-1/
