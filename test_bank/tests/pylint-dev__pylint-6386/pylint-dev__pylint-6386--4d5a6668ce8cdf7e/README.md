# test_short_verbose_does_not_consume_grouped_help_option

- **Instance:** `pylint-dev__pylint-6386`
- **Test ID:** `pylint-dev__pylint-6386--4d5a6668ce8cdf7e`
- **Status:** Accepted into the active test bank
- **Rule:** Candidate patches must fail in the recorded way
- **Introduced:** Round 2

## What it checks

Because `-v` is a no-argument flag, it must not consume a following option in a standard grouped-short-option invocation. Thus `-vh` must parse like `-v -h`.

## Expected behavior

Pylint prints its help text, including `usage: pylint [options]`, and exits successfully with status 0.

## Test command

`cd /testbed && python -m pytest -q tests/test_self.py::TestCallbackOptions::test_short_verbose_does_not_consume_grouped_help_option`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
- `negative_admission_decision.json`, when present: the separate decision about admitting the gold-patch failure.
