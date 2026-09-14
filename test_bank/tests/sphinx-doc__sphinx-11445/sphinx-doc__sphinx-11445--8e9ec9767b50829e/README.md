# test_rst_prolog_after_docinfo_with_inline_markup

- **Instance:** `sphinx-doc__sphinx-11445`
- **Test ID:** `sphinx-doc__sphinx-11445--8e9ec9767b50829e`
- **Status:** Accepted into the active test bank
- **Rule:** Candidate patches must pass
- **Introduced:** Round 1

## What it checks

Leading document metadata remains docinfo when its value contains reStructuredText inline markup. Classification must depend on the space after the field marker, not on whether the value contains backticks.

## Expected behavior

Sphinx collects the leading author field as document metadata with the externally observable value `The Sphinx team`; the prolog must be inserted after that field.

## Test command

`cd /testbed && python -m pytest -q tests/test_markup.py::test_rst_prolog_after_docinfo_with_inline_markup`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
