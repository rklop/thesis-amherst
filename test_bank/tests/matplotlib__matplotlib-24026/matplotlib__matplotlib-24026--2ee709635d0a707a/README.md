# test_stackplot_color_cycle_repeats

- **Instance:** `matplotlib__matplotlib-24026`
- **Test ID:** `matplotlib__matplotlib-24026--2ee709635d0a707a`
- **Status:** Accepted into the active test bank
- **Rule:** Candidate patches must pass
- **Introduced:** Round 2

## What it checks

The `colors` parameter is explicitly documented as a sequence “to be cycled through”; when it is shorter than the number of `y` series, colors repeat from the beginning. Preserving that behavior is part of fixing `CN` color handling without replacing cycling by one-to-one indexing.

## Expected behavior

The call returns three PolyCollections and their observable facecolors are red, blue, red in order.

## Test command

`cd /testbed && python -m pytest -q lib/matplotlib/tests/test_axes.py::test_stackplot_color_cycle_repeats`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
