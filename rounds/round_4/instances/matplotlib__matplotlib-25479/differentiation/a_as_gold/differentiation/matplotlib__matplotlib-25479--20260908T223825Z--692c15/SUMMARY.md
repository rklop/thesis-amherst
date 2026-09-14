# Differentiating-test run: `matplotlib__matplotlib-25479`

- Status: **completed**
- Image: `docker.io/swebench/sweb.eval.x86_64.matplotlib_1776_matplotlib-25479:latest`
- Run directory: `/home/rocky/SWE-ABS/mini-swe-agent/runs/spec-refinement-core-r2-negative-filter-on-v1/round-04/bidirectional-differentiation/instances/matplotlib__matplotlib-25479/a_as_gold/differentiation/matplotlib__matplotlib-25479--20260908T223825Z--692c15`
- Test: `test_registered_alias_survives_recreating_copy`
- Test command: `python -m pytest -q lib/matplotlib/tests/test_colormap_alias_copy.py`

## Specification gap

A registered alias must remain the effective colormap name after registry lookup copies the colormap. Renaming only the snapshot at registration is insufficient for Colormap subtypes whose copy() reconstructs an equivalent object with its intrinsic name.

## Input/output contract

Input: Register a ListedColormap subtype named "original_name" under "_test_registered_alias". Its public copy() reconstructs the same colormap. Pass the alias to pyplot.set_cmap, then create an image without an explicit cmap.

Expected output: Image creation succeeds, rcParams retains "_test_registered_alias", and the image's resolved colormap reports that alias.

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
