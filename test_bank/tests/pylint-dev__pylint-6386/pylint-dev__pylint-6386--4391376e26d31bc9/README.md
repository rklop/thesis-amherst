# test_verbose_help_does_not_require_value

- **Instance:** `pylint-dev__pylint-6386`
- **Test ID:** `pylint-dev__pylint-6386--4391376e26d31bc9`
- **Status:** Accepted into the active test bank
- **Rule:** Candidate patches must pass
- **Introduced:** Round 1

## What it checks

The short verbose option is not merely preprocessed as a flag; its public help synopsis must also declare zero arity and must not advertise a VERBOSE operand.

## Expected behavior

Help exits with status 0, and the verbose option's synopsis contains no VERBOSE metavar, showing that -v/--verbose takes no value.

## Test command

`cd /testbed && python -m pytest -q tests/test_self.py::TestCallbackOptions::test_verbose_help_does_not_require_value`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
