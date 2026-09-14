# test_set_cmap_uses_current_colormap_class_after_reload

- **Instance:** `matplotlib__matplotlib-25479`
- **Test ID:** `matplotlib__matplotlib-25479--3124f1099037cf13`
- **Status:** Accepted into the active test bank
- **Rule:** Candidate patches must pass
- **Introduced:** Round 2

## What it checks

`pyplot.set_cmap` must recognize instances of the currently loaded public `matplotlib.colors.Colormap` class, rather than relying on a stale class binding captured when `pyplot` was imported.

## Expected behavior

`set_cmap` completes without exception and `matplotlib.rcParams['image.cmap']` becomes `test_registered_alias`.

## Test command

`cd /testbed && python -m pytest -q lib/matplotlib/tests/test_pyplot.py::test_set_cmap_uses_current_colormap_class_after_reload`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
