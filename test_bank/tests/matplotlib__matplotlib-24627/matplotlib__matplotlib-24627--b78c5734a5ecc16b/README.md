# test_clf_deparents_artist_from_falsey_figure

- **Instance:** `matplotlib__matplotlib-24627`
- **Test ID:** `matplotlib__matplotlib-24627--b78c5734a5ecc16b`
- **Status:** Accepted into the active test bank
- **Rule:** Candidate patches must pass
- **Introduced:** Round 2

## What it checks

Clearing must unset an artist's parent references unconditionally, even when its valid Figure parent has false truthiness.

## Expected behavior

The cleared line has both line.axes is None and line.figure is None.

## Test command

`cd /testbed && python -m pytest -q lib/matplotlib/tests/test_axes.py::test_clf_deparents_artist_from_falsey_figure`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
