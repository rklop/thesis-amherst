# test_hermite_normal_tall_single_column_sign

- **Instance:** `sympy__sympy-23413`
- **Test ID:** `sympy__sympy-23413--62ccf1c1efb8f961`
- **Status:** Rejected and not used in later generation
- **Rule:** Candidate patches must fail in the recorded way
- **Introduced:** Round 2

## What it checks

For a tall integer matrix, rows above the bottom n rows still participate in canonical HNF sign normalization. In a one-column matrix, the column must be globally negated when its first nonzero entry is negative.

## Expected behavior

DM([[2], [-3]], ZZ): the equivalent column is sign-normalized so its first nonzero entry is positive. The generated candidate instead exits after processing the bottom row and leaves DM([[-2], [3]], ZZ).

## Test command

`cd /testbed && python bin/test sympy/polys/matrices/tests/test_normalforms.py -k test_hermite_normal_tall_single_column_sign`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
- `negative_admission_decision.json`, when present: the separate decision about admitting the gold-patch failure.
