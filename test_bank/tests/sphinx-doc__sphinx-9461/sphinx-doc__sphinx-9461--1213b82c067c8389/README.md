# test_class_property_uses_registered_attrgetter

- **Instance:** `sphinx-doc__sphinx-9461`
- **Test ID:** `sphinx-doc__sphinx-9461--1213b82c067c8389`
- **Status:** Accepted into the active test bank
- **Rule:** Candidate patches must pass
- **Introduced:** Round 2

## What it checks

Documenting a @classmethod/@property must continue to route attribute lookup through a type-specific getter registered with the public add_autodoc_attrgetter API.

## Expected behavior

Autodoc emits the class property directive and docstring, and the registered getter records lookup of Foo.classprop.

## Test command

`cd /testbed && python -m pytest -q tests/test_ext_autodoc_autoproperty.py::test_class_property_uses_registered_attrgetter`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
