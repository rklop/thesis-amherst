# test_abstract_class_property_signature_prefix

- **Instance:** `sphinx-doc__sphinx-9461`
- **Test ID:** `sphinx-doc__sphinx-9461--9c185e89e338b2ef`
- **Status:** Accepted into the active test bank
- **Rule:** Candidate patches must pass
- **Introduced:** Round 1

## What it checks

When a property is both class-level and abstract, its public signature should preserve the established modifier order: abstract before class, followed by property.

## Expected behavior

The externally visible signature text is exactly "abstract class property score".

## Test command

`cd /testbed && python -m pytest -q tests/test_domain_py_classproperty.py`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
