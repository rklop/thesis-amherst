# test_set_cmap_prefers_colormap_for_dual_type

- **Instance:** `matplotlib__matplotlib-25479`
- **Test ID:** `matplotlib__matplotlib-25479--dc7198fe2a3150e4`
- **Status:** Accepted into the active test bank
- **Rule:** Candidate patches must pass
- **Introduced:** Round 1

## What it checks

When an input satisfies both documented accepted types, str and Colormap, set_cmap must follow colormap-resolution precedence and use the Colormap.name as the default, not the object's unrelated string value.

## Expected behavior

plt.set_cmap succeeds, and the alternate public entry point mpl.colormaps.get_cmap(None) returns a colormap named "viridis" without raising an exception.

## Test command

`cd /testbed && python -m pytest -q lib/matplotlib/tests/test_cmap_alias.py`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
