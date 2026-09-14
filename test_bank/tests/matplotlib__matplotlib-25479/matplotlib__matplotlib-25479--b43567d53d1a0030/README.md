# test_set_cmap_runtime_type_hints

- **Instance:** `matplotlib__matplotlib-25479`
- **Test ID:** `matplotlib__matplotlib-25479--b43567d53d1a0030`
- **Status:** Rejected and not used in later generation
- **Rule:** Candidate patches must fail in the recorded way
- **Introduced:** Round 2

## What it checks

The public `pyplot.set_cmap` API advertises that its argument accepts either a `Colormap` or a registered-name string. That annotation should also be resolvable through Python's standard runtime type-introspection entry point.

## Expected behavior

Type-hint resolution succeeds and returns `matplotlib.colors.Colormap | str` for `cmap`. Candidate_a instead raises `NameError` because `Colormap` is absent from `pyplot`'s runtime globals.

## Test command

`python -m pytest -q lib/matplotlib/tests/test_pyplot.py::test_set_cmap_runtime_type_hints`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
- `negative_admission_decision.json`, when present: the separate decision about admitting the gold-patch failure.
