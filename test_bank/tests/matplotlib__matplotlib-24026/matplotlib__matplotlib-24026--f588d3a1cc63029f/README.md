# test_stackplot_explicit_color_preserves_alpha

- **Instance:** `matplotlib__matplotlib-24026`
- **Test ID:** `matplotlib__matplotlib-24026--f588d3a1cc63029f`
- **Status:** Accepted into the active test bank
- **Rule:** Candidate patches must pass
- **Introduced:** Round 1

## What it checks

The `colors` parameter accepts complete Matplotlib color specifications, so resolving colors—including CN aliases—must preserve an explicitly supplied alpha channel. The generated candidate converts every valid color to RGB and silently drops alpha.

## Expected behavior

The collection facecolor is `[[0.2, 0.4, 0.6, 0.25]]`. Candidate B instead produces an opaque facecolor with alpha `1.0`.

## Test command

`python -m pytest -q lib/matplotlib/tests/test_axes.py::test_stackplot_explicit_color_preserves_alpha`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
