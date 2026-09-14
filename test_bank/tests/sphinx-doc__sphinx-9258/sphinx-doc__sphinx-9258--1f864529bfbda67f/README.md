# test_info_field_list_pipe_union_preserves_abbreviated_xrefs

- **Instance:** `sphinx-doc__sphinx-9258`
- **Test ID:** `sphinx-doc__sphinx-9258--1f864529bfbda67f`
- **Status:** Accepted into the active test bank
- **Rule:** Candidate patches must pass
- **Introduced:** Round 2

## What it checks

Pipe-separated type fields must preserve Sphinx's per-reference `~` abbreviation semantics. Each union member remains an independent cross-reference whose displayed label omits its qualified prefix.

## Expected behavior

The rendered type is `First | Second`, with two class references targeting `package.First` and `package.Second` respectively.

## Test command

`cd /testbed && python -m pytest -q tests/test_domain_py.py::test_info_field_list_pipe_union_preserves_abbreviated_xrefs`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
