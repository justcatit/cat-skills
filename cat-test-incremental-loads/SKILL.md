---
name: cat-test-incremental-loads
description: >
  An incremental load (a watermark, a change log, a merge) must not lose, duplicate or
  miss rows, and it fails quietly: late-arriving rows, missed updates, deletes the target
  never hears about, time zones on the watermark, re-runs. Tests at three levels of effort,
  cheap counts per day after every load, the rows on a window, the full comparison once in
  a while. Obligatory reading before writing tests for any load that is not a full reload,
  and when a comparison of a continuously written source is red for no reason.
---

# Test incremental loads

A full load is right or obviously wrong; an incremental load is wrong quietly, and its own
log says succeeded. The tests look at the data on both sides. Every fact is on the
documentation page it links; the same page as markdown is at the URL with `index.md`
appended, which is the form to read when your fetch tool summarizes HTML.

## Three levels

Detect: counts per day over a window longer than the longest replication delay, after
every load. Detailed: the rows on a short window, with a key, so the message names them.
Full: the whole table, rarely, the only test that finds a row missed a year ago. The failure
modes, the three tests, and the balance between them:
https://docs.justcat.it/how-to-guides/test-patterns/test-incremental-loads/

The comparison mechanics the levels are built on, rungs 2 to 4 of the ladder:
https://docs.justcat.it/how-to-guides/test-patterns/compare-data-across-systems/

## Deletes, history tables, and systems that never stand still

Deleted rows show up as extra on the target with `sets match`; when the target is meant to
keep them, `contains` with the target as the superset and a key, plus a `set is empty` that
no staged row has an id the source never had. A source written continuously is cut on both
sides at the same moment in the past, in UTC. Both on the page above, with the two extra
tests that belong here: no duplicates from re-runs, and the watermark moved but not too far.
https://docs.justcat.it/reference/tests/expectations/contains/

## The traps

- Both windows cut with the same predicate on the same column semantics; a target that
  does not keep the source's `ModifiedAt` cannot be tested this way, and a source without a
  reliable `ModifiedAt` is the first finding.
- A window shorter than the replication delay never sees the late row.
- A threshold in local time on one side and UTC on the other reproduces the time-zone
  failure in the test itself.
- The full comparison is not the nightly test; it is the weekend or pre-release test.
