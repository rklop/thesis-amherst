# test_set_cmap_string_subclass_uses_registry_name

- **Instance:** `matplotlib__matplotlib-25479`
- **Test ID:** `matplotlib__matplotlib-25479--4a8dedf724f0a4ba`
- **Status:** Accepted into the active test bank
- **Rule:** Candidate patches must pass
- **Introduced:** Round 3

## What it checks

A `str` subclass passed to `pyplot.set_cmap` must follow the documented string-input behavior, even if it has an unrelated `.name` attribute. The string value is the registered colormap name.

## Expected behavior

`rcParams['image.cmap']` remains `string_subclass_cmap`, and the new image's colormap maps index 0 to red RGBA `(1, 0, 0, 1)`.

## Test command

`python -m pytest -q lib/matplotlib/tests/test_pyplot.py::test_set_cmap_string_subclass_uses_registry_name`

## Files

- `test.patch`: the executable test change.
- `proposal.json`: the generator's full test proposal.
- `details.json`: source round, direction, hashes, and admission metadata.
- `test_decision.json`: the recorded admission decision.
- `evidence/`: candidate patches, gold patch, and gold execution evidence.
