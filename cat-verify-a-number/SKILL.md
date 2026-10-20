---
name: cat-verify-a-number
description: >
  The user asks to verify one specific number or measure, or says it looks wrong, cannot be
  right, is not to be trusted: a total on a dashboard, a KPI in a report, a figure in an
  email, and either the project has no data sources yet or the request is clearly about
  that one number. Obligatory reading before touching such a
  request: the path from the number to the test that settles it, in order. Not for a general
  "help me with my data", and not for a request that already names a table, a test or a data
  source; those have skills of their own.
---

# Verify a number

One number somebody does not believe. The work is a path, in order; each step can end the
search, and a step skipped is the usual reason the test at the end proves nothing.
Every fact is on the documentation page it links; the same page as markdown is at the URL
with `index.md` appended, which is the form to read when your fetch tool summarizes HTML.
Before step 1, `catcli show` on the project when there is one: a data source already on the
system that shows the number skips the questions of step 1.

## 1. Where the number is

Find out which system shows the number and which system computes it; they are often two.
The user names things, not systems: "the dashboard", "the finance report", "the sheet from
controlling". Turn each into a system, with the one or two facts it needs, and check the user
can reach it at all:

- **Power BI**: the workspace and the name of the published model, or the `.pbix` file opened
  in Power BI Desktop on the machine that runs CAT. CAT queries the model in DAX, never the
  picture: https://docs.justcat.it/how-to-guides/data-platforms/power-bi/
- **A database or a warehouse** (SQL Server, Fabric, Databricks, Snowflake, Postgres): which
  server or workspace, which database or catalog, and how the user signs in. The provider
  behind each system: https://docs.justcat.it/reference/data-sources/technologies/
- **A sheet or a file**: where it is, as a path the run can reach, and which sheet:
  https://docs.justcat.it/how-to-guides/data-platforms/test-csv-and-excel-files/
- **A number in an email or a slide** has no system. Ask where it came from; that is the
  system.

Ask only for what the user can know, in one message, with your guess attached when you have
one ("I assume the dashboard reads the Sales model in the Finance workspace; is that
right?"). When the user cannot say what feeds the dashboard, say so and ask them to find out
from the report's owner; a number nobody can reach cannot be verified. Say what you are about
to do: map that system as a data source in the project and put tests on the number.

## 2. Create the data source and check it

One data source for the system that shows the number; load the data-sources skill to write
the entry and check the connection before anything else. A connection that fails here is
the first finding, not a detour.

## 3. Read the definition of the number

Where the number is computed, read how: the measure in the semantic model, the view or the
stored procedure, the step of the load. For a Power BI visual, Performance Analyzer hands
over the exact query behind it:
https://docs.justcat.it/how-to-guides/data-platforms/power-bi/get-the-dax-query-behind-a-visual/

Then run that definition once, as a query, not as a test, and compare with the number the
user sees. While reading, look for the usual problems and name any you find:

- a filter the user does not know about: a status excluded, a period boundary, a time zone,
  a currency, a default slicer on the page;
- gross against net: credit notes, returns, discounts counted on one side only;
- rows doubled by a join or a relationship, or lost by an inner join;
- the period not loaded at all, or loaded twice. A smoke test on the table behind the
  number for its period is the cheapest check and the first test worth saving:
  https://docs.justcat.it/how-to-guides/test-patterns/smoke-tests-with-set-is-not-empty/

When the query returns something else than the display, the report is bending the number
after the model computed it; a data test cannot see inside a report. Say so, pin what the
model gives (step 6), and leave the rest to the report author.

## 4. Where the number was loaded from

Ask the user, or confirm your guess, which system the data behind the number is loaded from:
the ERP, the ledger, the source database. That is the other end. When it is reachable, add
its data source the same way as in step 2. When it is not, the comparison is out of reach,
and the honest proof is a rule on the end you have: the number is not negative, the parts sum
to the total, net never exceeds gross, this month is within a distance of last month. Say
which kind of proof you are giving: consistency, not agreement with the source.

## 5. Compare the two ends, counts first

Load the compare-data-across-systems skill and climb its ladder from the cheap end: the one
number on both sides in one test, with a tolerance so rounding is not noise
(https://docs.justcat.it/reference/tests/tolerance/); when it fails, the same number by
month, region or account says where; then the rows. A failing total with passing groups is a
filter difference between the two ends, and the usual suspects are the same list as in
step 3: https://docs.justcat.it/how-to-guides/test-patterns/differences-between-systems/
With a Power BI model on one side:
https://docs.justcat.it/how-to-guides/data-platforms/power-bi/test-measures/

Both numbers are often right and were never the same number. Then the business picks the
definition, and the test keeps both sides honest.

## 5b. Go down the chain, one hop at a time

The chain is rarely two boxes. A member of the data team can usually recite it whole:
dashboard ← semantic model ← warehouse fact ← staging ← ERP, with a nightly load between
each pair. Take the whole chain when it is offered, and write it down. The ends are still
compared first, because when they agree the hops do not matter. When they disagree, the
hops say where: every hop is a data source and a counts-first comparison with its
neighbour, the same ladder as step 5, and the first hop that disagrees is where the number
breaks. Go as deep as the user can take you; a pair of hops the user cannot reach is a gap
you name, not a reason to stop above it. The hop that breaks gives the test of step 6; the
hops above it that agree are worth a smoke test each, so the next break is caught where it
happens.

## 6. The test that settles it

The first rung that disagrees, on the first hop that disagrees, is the permanent test, in a
suite named for what it proves; it
fails today with both numbers in the message and runs after every load from now on. When no
rung disagrees, or there is only one end, pin the number: the query that computes it plus the
condition that means wrong, expecting `set is empty`, so the next drift is caught:
https://docs.justcat.it/how-to-guides/test-patterns/my-expectation-does-not-exist/
A closed period is the classic pin.

## The traps

- A test written before the definition was read; it then proves the wrong number.
- A comparison attempted before both connections were checked.
- A CAT term the user never used (sets match, expectation, smoke test) without saying what it
  means in their words first.
- "I cannot verify it" when one end is reachable; a rule on that end is still a test.
