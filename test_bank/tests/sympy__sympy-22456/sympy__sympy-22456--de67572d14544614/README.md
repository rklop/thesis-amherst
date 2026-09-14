# test_String_args_do_not_make_text_symbolic

- **Instance:** `sympy__sympy-22456`
- **Test ID:** `sympy__sympy-22456--de67572d14544614`
- **Status:** Accepted into the active test bank
- **Rule:** Candidate patches must pass
- **Introduced:** Round 1

## What it checks

Positional reconstruction must preserve String as literal, non-symbolic data. Its text must not become a Symbol visible through Basic tree queries.

## Expected behavior

The reconstructed value equals the original String, and has(Symbol('foo')) returns False.

## Test command

`cd /testbed && python -c "from sympy.codegen.tests.test_ast import test_String_args_do_not_make_text_symbolic; test_String_args_do_not_make_text_symbolic()"`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
