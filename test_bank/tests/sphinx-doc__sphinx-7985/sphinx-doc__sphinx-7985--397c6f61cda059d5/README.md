# test_linkcheck_star_import_does_not_expose_os

- **Instance:** `sphinx-doc__sphinx-7985`
- **Test ID:** `sphinx-doc__sphinx-7985--397c6f61cda059d5`
- **Status:** Accepted into the active test bank
- **Rule:** Candidate patches must pass
- **Introduced:** Round 1

## What it checks

Adding local-link validation should not broaden the linkcheck module's implicit public import surface with an unrelated `os` name. The existing `path` binding is sufficient for the feature.

## Expected behavior

The resulting namespace does not contain `os`. candidate_b reuses the existing `path` import and passes; candidate_a introduces a top-level `os` import and fails.

## Test command

`cd /testbed && pytest -q tests/test_build_linkcheck.py::test_linkcheck_star_import_does_not_expose_os`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
