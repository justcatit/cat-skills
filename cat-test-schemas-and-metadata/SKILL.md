---
name: cat-test-schemas-and-metadata
description: >
  The structure must be right before the data can be: the tables the design says exist,
  the columns with the types the design says, the technical columns every table of a layer
  must carry. The design is a sheet, reality is a catalog view, both are data sources, so
  the structure is tested with the same five expectations. Obligatory reading before
  writing a test against INFORMATION_SCHEMA, a catalog, a design spreadsheet or a data
  dictionary, and before enforcing a naming or layering convention as a test.
---

# Test schemas and metadata

The design sheet is a data source (Excel or CSV), the platform's catalog is another, and
the test compares the two. Every fact is on the documentation page it links; the same page
as markdown is at the URL with `index.md` appended, which is the form to read when your
fetch tool summarizes HTML.

## The pattern

`contains` with the catalog as the first set: every designed column must exist, extra
technical columns may; `sets match` when nothing undesigned may exist either; a composite
key of schema, table and column so the message names the exact column and shows the
designed type next to the real one. Tables only first, as the cheaper check; mandatory
columns per layer as a template over the list of tables:
https://docs.justcat.it/how-to-guides/test-patterns/test-schemas-and-metadata/

The design sheet as a data source: https://docs.justcat.it/reference/data-sources/providers/excel-2/
https://docs.justcat.it/reference/data-sources/providers/csv-2/
The template for one test per table: https://docs.justcat.it/reference/tests/templates/
The expectation: https://docs.justcat.it/reference/tests/expectations/contains/

## The traps

- The sheet's types and the catalog's are spelled differently (`string` against
  `STRING`, `decimal(18,2)` against `DECIMAL(18,2)`); CAT ignores case, not spelling: a
  `LOWER()` or a mapping column in the sheet.
- A catalog query over every schema is large; filter to the layer under test, and key on
  schema, table, column so both sides sort the same.
- The catalog view differs per platform (`INFORMATION_SCHEMA`, `sys.columns`, Unity
  Catalog's `system.information_schema`); the data-sources skill has the provider's
  language.
