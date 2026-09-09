---
name: cat-set-up-data-source
description: >
  Add a data source to a CAT project, or create the project file first if there is none.
  Use when a test needs a database, file or model that the project does not know yet.
  Secrets never go into the project file: a connection string holds %ENV_VAR% placeholders
  and the values live in environment variables.
---

# Set up a data source

A data source is a named entry under `Data sources:` in the `.cat.yaml` project file. Tests
refer to it by that name. Every fact about data sources is in the documentation; this skill
only tells you what to do, in which order. Read a page by appending `index.md` to its URL.

## 1. Find or create the project file

- Look for a `*.cat.yaml` file in the working directory. If there is exactly one, that is the
  project. If there are several, ask which one.
- If there is none, create one: `catcli new` in the folder where the tests will live, then
  open the file it wrote. Details: https://docs.justcat.it/how-to-guides/organize-and-run-tests/how-to-create-project-file/

## 2. Check what the project already has

- Read the `Data sources:` list in the project file. Also follow every `Get list of data
  sources from:` entry, since data sources can come from other files or a database.
- If a data source already points at the same system, reuse it. Do not add a second one under
  a different name; names are not checked for duplicates and a test would pick one at random.

## 3. Choose the provider

- Pick the provider by the system you connect to. The table of what CAT can read, with the
  provider name and version to write: https://docs.justcat.it/reference/data-sources/providers/overview/
- Open the provider's own page before writing the connection string; the shape differs per
  provider and some take a file path or a folder instead. Index of provider pages:
  https://docs.justcat.it/reference/data-sources/providers/
- If the system is Power BI, Databricks, Microsoft Fabric, Dataverse, or CSV and Excel files,
  read the platform guide first, it says what to connect to: https://docs.justcat.it/how-to-guides/data-platforms/

## 4. Write the block

- Add one entry to `Data sources:` with `Name`, `Provider`, `Connection string`. All
  properties and their synonyms: https://docs.justcat.it/reference/data-sources/properties/
- Put every secret behind a `%NAME%` placeholder and tell the user which environment
  variables to set. Never write a password, token or full connection string into the file.
  The rule and the ways around secrets (integrated authentication, Entra ID):
  https://docs.justcat.it/how-to-guides/organize-and-run-tests/work-with-secrets/
- How placeholders are resolved, and how to make sure CAT sees the variable:
  https://docs.justcat.it/reference/project-file/environment-variables/

Example, in the documentation's spelling:

```yaml
Data sources:
- Name: FlightsSystem
  Provider: SqlServer@2
  Connection string: >
    Server=sql-aero-flights-prod.database.windows.net;
    Database=sqldb-aero-flights-prod;
    User Id=%FLIGHTS_TESTING_PRINCIPAL_ID%;
    Password=%FLIGHTS_TESTING_PRINCIPAL_SECRET%;
```

## 5. Check that it works

- Run one cheap command through it:
  `catcli exec -d <name> -c "SELECT 1"` (or the provider's equivalent of a trivial query).
- Exit code 3 means the CLI is not signed in and has no license key: load the `cat-install-cat`
  skill. Any other error names the provider's problem; fix the connection string, not the test.
- Then continue with the `cat-write-tests` skill.
