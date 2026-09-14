# test_catalog_deduplicates_locations_without_reordering

- **Instance:** `sphinx-doc__sphinx-10466`
- **Test ID:** `sphinx-doc__sphinx-10466--96d16b0e78d8c06c`
- **Status:** Accepted into the active test bank
- **Rule:** Candidate patches must fail in the recorded way
- **Introduced:** Round 1

## What it checks

Deduplicating Message.locations must preserve the order in which each unique location was first encountered; it must not sort the locations.

## Expected behavior

Catalog iteration produces one Message with locations exactly [('z-first.rst', 42), ('a-second.rst', 7)]: the duplicate is removed without reordering the two unique locations.

## Test command

`python -m pytest -q tests/test_build_gettext_locations.py`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
- `negative_admission_decision.json`, when present: the separate decision about admitting the gold-patch failure.
