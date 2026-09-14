# test_stackplot_color_iterator_does_not_advance_prop_cycle

- **Instance:** `matplotlib__matplotlib-24026`
- **Test ID:** `matplotlib__matplotlib-24026--35a69adfca66c668`
- **Status:** Accepted into the active test bank
- **Rule:** Candidate patches must pass
- **Introduced:** Round 2

## What it checks

The explicit `colors` input should continue to accept one-shot iterables, including when they contain `CN` aliases. Normalizing those colors must not alter or advance the Axes property cycle.

## Expected behavior

Stackplot completes without error; its two returned areas are green and blue, respectively, while the subsequently plotted line is red (C0), showing that stackplot did not consume or replace the Axes cycle.

## Test command

`python -m pytest -q lib/matplotlib/tests/test_axes.py::test_stackplot_color_iterator_does_not_advance_prop_cycle`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
