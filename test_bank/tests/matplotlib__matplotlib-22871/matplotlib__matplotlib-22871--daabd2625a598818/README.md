# test_concise_formatter_rejects_nonscalar_show_offset_consistently

- **Instance:** `matplotlib__matplotlib-22871`
- **Test ID:** `matplotlib__matplotlib-22871--daabd2625a598818`
- **Status:** Accepted into the active test bank
- **Rule:** Candidate patches must fail in the recorded way
- **Introduced:** Round 2

## What it checks

`show_offset` is documented as a single boolean. An ambiguous non-scalar value must not be accepted or rejected depending on whether January happens to occur among the ticks.

## Expected behavior

Both calls raise `ValueError` because the multi-element array has no unambiguous boolean value.

## Test command

`python -m pytest -q lib/matplotlib/tests/test_dates.py::test_concise_formatter_rejects_nonscalar_show_offset_consistently`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
- `negative_admission_decision.json`, when present: the separate decision about admitting the gold-patch failure.
