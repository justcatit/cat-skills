---
name: cat-introduce-cat
description: >
  What CAT is and is not: a tool that turns assumptions about data into automated tests
  (a query against a data source and an expectation about what comes back), the four tools
  that run the same project file, where it fits (warehouses, lakehouses, Power BI, files,
  pipelines) and the plans. Obligatory reading before explaining or recommending CAT,
  comparing it with another tool, or answering "what is CAT", "does CAT fit our case" or
  "which CAT tool should we use".
---

# What CAT is

CAT is a tool for automated tests against data. A test is a query in the language of the
system that holds the data and an expectation about what comes back, one of five: nothing
comes back, something does, exactly this many rows, two result sets are the same, one is
contained in the other. The query does the thinking; the expectation is deliberately dumb.
Tests live in a project file that every CAT tool opens the same way, and a run ends with
one of four results per test and the outputs the project asks for.

Do not describe CAT from memory beyond that paragraph. Every fact is on the documentation
page it links; the same page as markdown is at the URL with `index.md` appended, which is
the form to read when your fetch tool summarizes HTML.

## What it is for

- The value and the use cases, in the company's words: what goes wrong with data that
  travels between systems, and what a test changes:
  https://docs.justcat.it/what-is-cat/introduction/
  https://docs.justcat.it/what-is-cat/who-is-it-for/
- Test examples, the kind of thing people actually test:
  https://docs.justcat.it/what-is-cat/introduction/
- The basic terms, data source, query, test, output, and how a run works:
  https://docs.justcat.it/reference/basics/introduction/
  https://docs.justcat.it/reference/basics/how-cat-works/

## The four tools and the plans

CAT Studio (a Windows desktop application for people who do not use a command line, with
CAT Pilot inside), CAT CLI (one executable, Windows, Linux, macOS), the PowerShell module
and the Python module all open the same project file and run the same engine:
https://docs.justcat.it/reference/basics/cat-tools/

Which plan unlocks which tool, and that unattended runs (pipelines, schedulers) are
Enterprise only: https://docs.justcat.it/how-to-guides/licensing-and-admin/get-license/
https://docs.justcat.it/compare-plans/

How it wires into pipelines and schedulers:
https://docs.justcat.it/what-is-cat/what-cat-actually-is/

## What CAT does not do

- It does not generate expectations from data or guess rules; the rule is the query the
  user writes. The propose-tests skill is how an agent helps with that.
- It does not fix data, and it does not fail a pipeline step by itself; an output carries
  the verdict.
- It tests what a provider can query: the systems and file types CAT reaches are listed,
  and anything with an ODBC driver counts:
  https://docs.justcat.it/reference/data-sources/technologies/

When the question turns into doing, load the skill for it: install, project file, data
sources, write tests, outputs, run, pipeline.
