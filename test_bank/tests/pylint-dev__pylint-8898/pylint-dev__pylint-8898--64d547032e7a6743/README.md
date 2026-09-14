# test_ignore_paths_regex_with_comma_in_quantifier

- **Instance:** `pylint-dev__pylint-8898`
- **Test ID:** `pylint-dev__pylint-8898--64d547032e7a6743`
- **Status:** Rejected and not used in later generation
- **Rule:** Candidate patches must fail in the recorded way
- **Introduced:** Round 1

## What it checks

Comma-aware regex parsing must also apply to the public `--ignore-paths` option. A comma inside a valid quantifier is part of that path regex, not a separator between path regexes.

## Expected behavior

Pylint preserves `{1,3}` as part of one regex, ignores the matching file, emits no lint diagnostics, and exits with status 0.

## Test command

`cd /testbed && python -m pytest -q tests/config/test_argparse_config.py::TestArgparseOptionsProviderMixin::test_ignore_paths_regex_with_comma_in_quantifier`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
- `negative_admission_decision.json`, when present: the separate decision about admitting the gold-patch failure.
