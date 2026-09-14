# test_info_field_list_rtype_union

- **Instance:** `sphinx-doc__sphinx-9258`
- **Test ID:** `sphinx-doc__sphinx-9258--44d6313f6191ab73`
- **Status:** Accepted into the active test bank
- **Rule:** Candidate patches must pass
- **Introduced:** Round 1

## What it checks

Pipe-separated unions in the public `:rtype:` information field must create an independently linkable cross-reference for each member, just as other Python type fields do.

## Expected behavior

The rendered return type displays `bytes | str` and contains separate internal hyperlinks to the `bytes` and `str` class definitions.

## Test command

`cd /testbed && pytest -q tests/test_domain_py.py::test_info_field_list_rtype_union`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
