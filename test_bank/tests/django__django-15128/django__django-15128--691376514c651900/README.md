# test_combine_or_then_deeper_self_relation

- **Instance:** `django__django-15128`
- **Test ID:** `django__django-15128--691376514c651900`
- **Status:** Accepted into the active test bank
- **Rule:** Candidate patches must pass
- **Introduced:** Round 1

## What it checks

An OR-combined QuerySet must remain safely composable with subsequent related-field filters, including filters that require additional aliases for repeated self-joins.

## Expected behavior

The final QuerySet evaluates without an alias collision and contains only the leaf Tag whose fourth ancestor is root.

## Test command

`./tests/runtests.py queries.tests.Queries4Tests.test_combine_or_then_deeper_self_relation`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
