# test_rst_prolog_preserves_valueless_docinfo

- **Instance:** `sphinx-doc__sphinx-11445`
- **Test ID:** `sphinx-doc__sphinx-11445--473539d1f87a515e`
- **Status:** Accepted into the active test bank
- **Rule:** Candidate patches must pass
- **Introduced:** Round 1

## What it checks

A nonempty rst_prolog must be inserted after leading valueless document metadata such as :orphan:, not before it. Valueless docinfo remains metadata even though no space follows its second colon.

## Expected behavior

The build environment records app.env.metadata['restructuredtext']['orphan'] as the empty string, showing that :orphan: remained document metadata rather than becoming body content.

## Test command

`cd /testbed && python -m pytest -q tests/test_markup.py::test_rst_prolog_preserves_valueless_docinfo`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
