---
name: cat-test-compliance-rules
description: >
  Compliance rules as data tests with the run as the evidence: personal columns masked for
  flagged customers, an erased person gone from every table that ever held them, nothing
  older than the retention period, consent respected. The same violation-query shape as any
  rule, with the message kept free of the data it protects. Obligatory reading before
  writing a test about erasure, masking, retention, consent or any rule with a legal
  deadline, and before putting personal data into a failure message.
---

# Test compliance rules

A compliance rule is a data rule with a legal deadline; it tests like any other rule, as a
query that finds the violations, and the run is the evidence. Every fact is on the
documentation page it links; the same page as markdown is at the URL with `index.md`
appended, which is the form to read when your fetch tool summarizes HTML.

## The pattern

Masked values for flagged customers, forgotten everywhere (one query when the erasure list
and the data are reachable from one place, `contains` with the masked IDs as the superset
when they are two systems), retention, consent; each with its test:
https://docs.justcat.it/how-to-guides/test-patterns/test-compliance-rules/

The violation-query shape it is built on:
https://docs.justcat.it/how-to-guides/test-patterns/find-problems-with-set-is-empty/
The expectations: https://docs.justcat.it/reference/tests/expectations/set-is-empty/
https://docs.justcat.it/reference/tests/expectations/contains/

## Keep the data out of the evidence

`Maximum errors logged: 0`: the offending row is personal data, so the message says how
many rows broke the rule and not what they contain; `Log number of errors: true` adds the
total. The outputs that carry the message go to the same places the data may not; the
outputs skill says which fields each output writes:
https://docs.justcat.it/reference/tests/failure-message/

## The traps

- A test that shows the erased person's e-mail in its failure message is itself a breach;
  the default sample of one row is not zero.
- "Gone" and "masked" are different rules with different queries; test for the marker the
  anonymization job actually writes (`--removed--`, a hash, `NULL`).
- The erasure list lives in one place; every other system is checked against it, not
  against its own flag.
