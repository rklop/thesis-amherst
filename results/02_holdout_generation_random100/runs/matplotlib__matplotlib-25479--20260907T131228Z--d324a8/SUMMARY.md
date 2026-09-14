# Differentiating-test run: `matplotlib__matplotlib-25479`

- Status: **completed**
- Image: `docker.io/swebench/sweb.eval.x86_64.matplotlib_1776_matplotlib-25479:latest`
- Run directory: `/home/rocky/SWE-ABS/mini-swe-agent/evaluation_records/differentiating_gold_pass_current_candidate_fail_random100_20260904/runs/matplotlib__matplotlib-25479--20260907T131228Z--d324a8`
- Test: `test_lookup_does_not_reassign_matching_cmap_name`
- Test command: `cd /testbed && python -m pytest -q lib/matplotlib/tests/test_colormap_registry_alias.py::test_lookup_does_not_reassign_matching_cmap_name`

## Specification gap

Registry lookup is a read operation and should not require reassigning a colormap's name when it already matches the registered key. Valid Colormap subclasses may expose an immutable name after construction.

## Input/output contract

Input: Register a ListedColormap subclass whose name permits initialization but rejects later reassignment, using its existing name as the registry key. Retrieve it through the public matplotlib.colormaps mapping and evaluate color index 0.

Expected output: Lookup succeeds, the returned colormap retains the registered name, and evaluating index 0 returns opaque red `(1.0, 0.0, 0.0, 1.0)`. Candidate A instead unconditionally assigns `cmap.name` during every lookup and raises AttributeError; candidate B only updates the stored snapshot when the registration name differs, so this matching-name case succeeds.

## Execution

| Candidate | Ran | Passed | Exit |
| --- | --- | --- | --- |
| candidate_a | True | False | 1 |
| candidate_b | True | True | 0 |

## MiniMax judgment

- Rating: **high_signal**
- Confidence: 0.95
- Summary: The test successfully differentiates between candidates by exposing candidate A's overbroad mutation of cmap.name during every registry lookup, versus candidate B's more surgical approach that only updates the name when it differs from the registered key. The test uses a custom ReadOnlyNameColormap to simulate a realistic edge case where a Colormap subclass exposes an immutable name property, which is plausible given that the public API accepts any Colormap subclass. Candidate B correctly passes because it avoids attempting to set the name when it already matches, while candidate A unconditionally assigns cmap.name = item and raises AttributeError.

## Main artifacts

- `selected_test.patch`
- `selected_proposal.json`
- `02_execution/selected/result.json`
- `03_minimax/verdict.json`
- `pipeline.log`
