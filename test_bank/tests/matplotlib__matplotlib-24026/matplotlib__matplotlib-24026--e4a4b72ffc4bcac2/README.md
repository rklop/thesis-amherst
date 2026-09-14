# test_stackplot_colors_preserve_prop_cycle

- **Instance:** `matplotlib__matplotlib-24026`
- **Test ID:** `matplotlib__matplotlib-24026--e4a4b72ffc4bcac2`
- **Status:** Accepted into the active test bank
- **Rule:** Candidate patches must pass
- **Introduced:** Round 1

## What it checks

Supplying explicit colors to stackplot must affect only the stacked areas; it must neither replace nor advance the Axes line property cycle, including when that cycle has already been partially consumed.

## Expected behavior

The line created after stackplot has the next preconfigured cycle color, magenta. The supplied gold patch preserves that state; the generated candidate replaces the Axes cycle with the stack colors, so the assertion fails.

## Test command

`cd /testbed && python -m pytest -q lib/matplotlib/tests/test_axes.py::test_stackplot_colors_preserve_prop_cycle`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
