# test_clf_deparents_direct_figure_artist

- **Instance:** `matplotlib__matplotlib-24627`
- **Test ID:** `matplotlib__matplotlib-24627--28aad9ea27bbe87d`
- **Status:** Rejected and not used in later generation
- **Rule:** Candidate patches must fail in the recorded way
- **Introduced:** Round 1

## What it checks

`clf()` must unset both parent attributes for artists owned directly by the Figure through `Figure.add_artist`, not only for artists found among an Axes' children.

## Expected behavior

The retained artist is deparented: both `artist.axes` and `artist.figure` are `None`.

## Test command

`cd /testbed && python -m pytest -q lib/matplotlib/tests/test_figure.py::test_clf_deparents_direct_figure_artist`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
- `negative_admission_decision.json`, when present: the separate decision about admitting the gold-patch failure.
