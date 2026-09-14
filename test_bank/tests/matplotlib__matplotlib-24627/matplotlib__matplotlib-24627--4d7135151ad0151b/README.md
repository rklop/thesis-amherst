# test_cla_deparents_artists_not_containers

- **Instance:** `matplotlib__matplotlib-24627`
- **Test ID:** `matplotlib__matplotlib-24627--4d7135151ad0151b`
- **Status:** Accepted into the active test bank
- **Rule:** Candidate patches must pass
- **Introduced:** Round 3

## What it checks

Axes.cla() must unset parent references on deparented Artists, but Containers are non-Artist grouping objects and must not gain Artist-only axes or figure attributes during clearing.

## Expected behavior

The Rectangle has axes and figure set to None, while the BarContainer still has no axes or figure attributes.

## Test command

`python -m pytest -q lib/matplotlib/tests/test_axes.py::test_cla_deparents_artists_not_containers`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
