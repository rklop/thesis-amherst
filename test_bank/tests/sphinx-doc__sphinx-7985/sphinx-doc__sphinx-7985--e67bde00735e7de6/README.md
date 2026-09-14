# test_existing_local_link_uses_real_filesystem

- **Instance:** `sphinx-doc__sphinx-7985`
- **Test ID:** `sphinx-doc__sphinx-7985--e67bde00735e7de6`
- **Status:** Rejected and not used in later generation
- **Rule:** Candidate patches must fail in the recorded way
- **Introduced:** Round 1

## What it checks

Local-link classification should reflect whether the target actually exists under the Sphinx source directory, without coupling the new check to the builder module's mutable legacy path helper.

## Expected behavior

The linkcheck build writes an `output.json` row for `target.dat` with status `working` and empty `info`, because the referenced source-tree file exists.

## Test command

`cd /testbed && python -m pytest -q tests/test_build_linkcheck_local.py::test_existing_local_link_uses_real_filesystem`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
- `negative_admission_decision.json`, when present: the separate decision about admitting the gold-patch failure.
