# test_ignore_paths_keeps_separator_after_literal_brace

- **Instance:** `pylint-dev__pylint-8898`
- **Test ID:** `pylint-dev__pylint-8898--3ad7455e4b17fe14`
- **Status:** Accepted into the active test bank
- **Rule:** Candidate patches must pass
- **Introduced:** Round 1

## What it checks

The new brace-aware parsing applies to ordinary regex-list options such as `bad-name-rgxs`, but must not change the existing comma-separated contract of the separate `--ignore-paths` option. A `{` that does not begin a quantifier is a valid literal regex character and must not hide a subsequent CSV separator.

## Expected behavior

The resulting ignore-path patterns independently match `foo{` and `second`. The designated gold retains this behavior, while the generated candidate merges them into one pattern because it treats every opening brace as an unclosed quantifier.

## Test command

`python -m pytest -q tests/config/test_argparse_config.py::TestArguments::test_ignore_paths_keeps_separator_after_literal_brace`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
