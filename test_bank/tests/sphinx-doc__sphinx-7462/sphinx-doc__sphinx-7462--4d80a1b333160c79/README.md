# test_unparse_empty_tuple_proxy

- **Instance:** `sphinx-doc__sphinx-7462`
- **Test ID:** `sphinx-doc__sphinx-7462--4d80a1b333160c79`
- **Status:** Accepted into the active test bank
- **Rule:** Candidate patches must pass
- **Introduced:** Round 1

## What it checks

Once an AST-compatible value has been recognized as a tuple, an empty `elts` field should be handled within that dispatch and produce the canonical empty-tuple representation without requiring a second tuple-type check.

## Expected behavior

The public unparser returns the exact string `()`. Candidate_a remains in the tuple branch and returns it; candidate_b re-checks the type after reading `elts` and raises `NotImplementedError`.

## Test command

`python -m pytest -q tests/test_pycode_ast.py::test_unparse_empty_tuple_proxy`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
