# Differentiating-test run: `matplotlib__matplotlib-25479`

- Status: **completed**
- Image: `docker.io/swebench/sweb.eval.x86_64.matplotlib_1776_matplotlib-25479:latest`
- Run directory: `/home/rocky/SWE-ABS/mini-swe-agent/runs/spec-refinement-core-r2-negative-filter-on-v1/round-03/bidirectional-differentiation/instances/matplotlib__matplotlib-25479/a_as_gold/differentiation/matplotlib__matplotlib-25479--20260908T174456Z--451568`
- Test: `test_set_cmap_string_subclass_uses_registry_name`
- Test command: `python -m pytest -q lib/matplotlib/tests/test_pyplot.py::test_set_cmap_string_subclass_uses_registry_name`

## Specification gap

A `str` subclass passed to `pyplot.set_cmap` must follow the documented string-input behavior, even if it has an unrelated `.name` attribute. The string value is the registered colormap name.

## Input/output contract

Input: Register a one-color red colormap under a `str` subclass whose value is `string_subclass_cmap` but whose `.name` attribute is `viridis`; pass that object to `plt.set_cmap`, then create an image.

Expected output: `rcParams['image.cmap']` remains `string_subclass_cmap`, and the new image's colormap maps index 0 to red RGBA `(1, 0, 0, 1)`.

## Execution

| Candidate | Ran | Passed | Exit |
| --- | --- | --- | --- |
| candidate_a | True | True | 0 |
| candidate_b | True | False | 1 |

## Main artifacts

- `selected_test.patch`
- `selected_proposal.json`
- `02_execution/selected/result.json`
- `03_minimax/verdict.json`
- `pipeline.log`
