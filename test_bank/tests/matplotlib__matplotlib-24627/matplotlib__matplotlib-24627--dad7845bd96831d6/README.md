# test_clear_fully_removes_artist

- **Instance:** `matplotlib__matplotlib-24627`
- **Test ID:** `matplotlib__matplotlib-24627--dad7845bd96831d6`
- **Status:** Rejected and not used in later generation
- **Rule:** Candidate patches must fail in the recorded way
- **Introduced:** Round 2

## What it checks

Axes.clear() must complete the normal Artist.remove() lifecycle, not merely set the artist's parent attributes to None. A subsequent remove() is therefore a repeated removal.

## Expected behavior

After clear(), line.axes and line.figure are None, and the subsequent remove() raises ValueError because the artist has already been removed.

## Test command

`python -m pytest -q lib/matplotlib/tests/test_axes.py::test_clear_fully_removes_artist`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
- `negative_admission_decision.json`, when present: the separate decision about admitting the gold-patch failure.
