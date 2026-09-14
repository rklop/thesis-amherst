# test_csv_regex_separator_after_literal_braces

- **Instance:** `pylint-dev__pylint-8898`
- **Test ID:** `pylint-dev__pylint-8898--62786e24781f2f8d`
- **Status:** Accepted into the active test bank
- **Rule:** Candidate patches must pass
- **Introduced:** Round 2

## What it checks

A comma must remain a list separator after a valid regex containing literal braces, even when `{` occurs outside a character class and `}` occurs inside one. Brace tracking must not collapse the following regex into the preceding entry.

## Expected behavior

Pylint emits `C0104: Disallowed name "bar" (disallowed-name)`, proving that `bar` was parsed as the second bad-name regex.

## Test command

`python -m pytest -q tests/config/test_config.py::test_csv_regex_separator_after_literal_braces`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
