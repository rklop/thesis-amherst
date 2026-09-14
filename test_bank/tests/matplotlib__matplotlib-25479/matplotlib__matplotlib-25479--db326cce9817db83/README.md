# test_set_cmap_str_subclass_uses_registered_name

- **Instance:** `matplotlib__matplotlib-25479`
- **Test ID:** `matplotlib__matplotlib-25479--db326cce9817db83`
- **Status:** Rejected and not used in later generation
- **Rule:** Candidate patches must fail in the recorded way
- **Introduced:** Round 1

## What it checks

When pyplot.set_cmap receives a valid string-like registered name, image.cmap must retain that registered name rather than an unrelated Colormap.name attribute on the input object.

## Expected behavior

plt.set_cmap completes and matplotlib.rcParams["image.cmap"] equals "swe_bench_registered_alias", not "viridis".

## Test command

`cd /testbed && python -m pytest -q lib/matplotlib/tests/test_pyplot_cmap_alias.py::test_set_cmap_str_subclass_uses_registered_name`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
- `negative_admission_decision.json`, when present: the separate decision about admitting the gold-patch failure.
