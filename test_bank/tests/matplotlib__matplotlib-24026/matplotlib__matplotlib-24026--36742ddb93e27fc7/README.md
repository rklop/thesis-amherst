# test_stackplot_rejects_single_color_string

- **Instance:** `matplotlib__matplotlib-24026`
- **Test ID:** `matplotlib__matplotlib-24026--36742ddb93e27fc7`
- **Status:** Accepted into the active test bank
- **Rule:** Candidate patches must fail in the recorded way
- **Introduced:** Round 3

## What it checks

The `colors` parameter is documented as a sequence of colors, not a single color value. A scalar color string such as `"red"` must therefore remain invalid rather than being silently accepted as a one-element color sequence.

## Expected behavior

The public call raises `ValueError`. The targeted pytest command exits with status 0 only when that rejection occurs.

## Test command

`python -m pytest -q lib/matplotlib/tests/test_axes.py::test_stackplot_rejects_single_color_string`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
- `negative_admission_decision.json`, when present: the separate decision about admitting the gold-patch failure.
