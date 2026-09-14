# test_bad_name_regex_csv_separator_after_literal_closing_brace

- **Instance:** `pylint-dev__pylint-8898`
- **Test ID:** `pylint-dev__pylint-8898--77454acfb8d3f993`
- **Status:** Accepted into the active test bank
- **Rule:** Candidate patches must pass
- **Introduced:** Round 2

## What it checks

A valid regex may contain a literal unmatched closing brace. That brace must not prevent a subsequent top-level comma from separating the next regex in the comma-separated option.

## Expected behavior

Pylint emits `Disallowed name "first"` and `Disallowed name "target_name"`. Candidate A preserves zero brace depth after the literal `}` and parses two regexes. Candidate B decrements brace depth below zero, parses one combined regex, and emits neither diagnostic.

## Test command

`cd /testbed && python -m pytest -q tests/config/test_regexp_csv.py::test_bad_name_regex_csv_separator_after_literal_closing_brace`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
