# test_autodata_class_property

- **Instance:** `sphinx-doc__sphinx-9461`
- **Test ID:** `sphinx-doc__sphinx-9461--b4da70c176c3e9a9`
- **Status:** Accepted into the active test bank
- **Rule:** Candidate patches must pass
- **Introduced:** Round 3

## What it checks

A classmethod-wrapped property must expose its getter docstring not only through automatic property-member discovery, but also when addressed explicitly through the public `autodata` directive.

## Expected behavior

The generated autodoc content contains `Class property documentation.` Candidate_a unwraps the descriptor for `DataDocumenter`; candidate_b leaves it as a Python 3.9 `classmethod`, whose visible docstring is not the wrapped property's docstring.

## Test command

`python -m pytest -q tests/test_ext_autodoc_autodata.py::test_autodata_class_property`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
