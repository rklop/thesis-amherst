# test_cla_deparents_container_children

- **Instance:** `matplotlib__matplotlib-24627`
- **Test ID:** `matplotlib__matplotlib-24627--e5acddf8d5aa20f8`
- **Status:** Accepted into the active test bank
- **Rule:** Candidate patches must pass
- **Introduced:** Round 1

## What it checks

Axes.cla() must deparent artists produced by container-returning plotting methods without failing during clear; this narrowly extends the reported plain-line behavior to Axes.bar rectangles.

## Expected behavior

ax.cla() completes without an exception, and every retained Rectangle has both axes and figure set to None.

## Test command

`cd /testbed && python -m pytest -q lib/matplotlib/tests/test_axes.py::test_cla_deparents_container_children`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
