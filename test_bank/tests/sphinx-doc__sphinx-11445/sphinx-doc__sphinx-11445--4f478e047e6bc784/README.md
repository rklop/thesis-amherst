# test_rst_prolog_preserves_colon_delimited_heading

- **Instance:** `sphinx-doc__sphinx-11445`
- **Test ID:** `sphinx-doc__sphinx-11445--4f478e047e6bc784`
- **Status:** Accepted into the active test bank
- **Rule:** Candidate patches must pass
- **Introduced:** Round 2

## What it checks

A leading `:name:` prefix is document metadata only when its closing colon is followed by whitespace or end-of-line. Without that separator, the line remains ordinary reStructuredText content and must stay adjacent to a following heading underline.

## Expected behavior

The generated index page contains a link to `colon-heading.html` whose visible caption is `:RFC:2822`.

## Test command

`cd /testbed && python -m pytest -q tests/test_markup.py::test_rst_prolog_preserves_colon_delimited_heading`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
