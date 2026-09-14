# test_short_verbose_value_matches_long_alias_error

- **Instance:** `pylint-dev__pylint-6386`
- **Test ID:** `pylint-dev__pylint-6386--a80d27e690143767`
- **Status:** Rejected and not used in later generation
- **Rule:** Candidate patches must fail in the recorded way
- **Introduced:** Round 1

## What it checks

The short `-v` spelling is an alias of `--verbose`, so invalid inline values should be validated and diagnosed using the same canonical option identity as the long spelling.

## Expected behavior

Both invocations exit with status 32 and write exactly `Option --verbose doesn't expects a value\n` to stderr.

## Test command

`cd /testbed && python -m pytest -q tests/config/test_verbose_short_option.py::test_short_verbose_value_matches_long_alias_error`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
- `negative_admission_decision.json`, when present: the separate decision about admitting the gold-patch failure.
