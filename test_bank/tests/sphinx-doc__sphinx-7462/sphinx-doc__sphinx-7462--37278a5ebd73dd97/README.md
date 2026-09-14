# test_unparse_deferred_ast_kind

- **Instance:** `sphinx-doc__sphinx-7462`
- **Test ID:** `sphinx-doc__sphinx-7462--37278a5ebd73dd97`
- **Status:** Rejected and not used in later generation
- **Rule:** Candidate patches must fail in the recorded way
- **Introduced:** Round 1

## What it checks

When reading a lazy AST node's empty `elts` materializes it as a non-tuple node, `unparse` should render the materialized node kind rather than committing to the earlier tuple view.

## Expected behavior

`unparse` returns `frozenset()`. Candidate_b re-evaluates the node kind after observing empty tuple elements; candidate_a remains committed to the tuple branch and returns `()`.

## Test command

`pytest -q tests/test_pycode_ast.py::test_unparse_deferred_ast_kind`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
- `negative_admission_decision.json`, when present: the separate decision about admitting the gold-patch failure.
