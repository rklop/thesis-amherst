# test_catalog_locations_are_unique_and_sorted

- **Instance:** `sphinx-doc__sphinx-10466`
- **Test ID:** `sphinx-doc__sphinx-10466--2e166d9dc5d7ca21`
- **Status:** Accepted into the active test bank
- **Rule:** Candidate patches must pass
- **Introduced:** Round 1

## What it checks

Deduplicated gettext locations must also have a deterministic canonical order, independent of the order in which document origins are encountered.

## Expected behavior

The yielded message has exactly [('alpha.rst', 3), ('alpha.rst', 20), ('zeta.rst', 7)] as its locations: duplicates are removed and locations are sorted by source and line.

## Test command

`cd /testbed && python -m pytest -q tests/test_build_gettext.py::test_catalog_locations_are_unique_and_sorted`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
