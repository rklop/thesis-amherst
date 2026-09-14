# Differentiating-test run: `scikit-learn__scikit-learn-14983`

- Status: **completed**
- Image: `docker.io/swebench/sweb.eval.x86_64.scikit-learn_1776_scikit-learn-14983:latest`
- Run directory: `/home/rocky/SWE-ABS/mini-swe-agent/runs/spec-refinement-core-r2-negative-filter-on-v1/round-02/bidirectional-differentiation/instances/scikit-learn__scikit-learn-14983/b_as_gold/differentiation/scikit-learn__scikit-learn-14983--20260908T133154Z--59cabf`
- Test: `test_repeated_split_repr_preserves_forwarded_cvargs`
- Test command: `cd /testbed && python -m pytest -q sklearn/model_selection/tests/test_repeated_split_repr.py`

## Specification gap

A repeated splitter's repr must preserve every explicit constructor parameter forwarded to its wrapped cross-validator, not only `n_splits`.

## Input/output contract

Input: A valid `_RepeatedSplits` subclass wraps `ShuffleSplit` through a compatible factory and is constructed with `test_size=0.25`, `n_repeats=3`, and `random_state=7`.

Expected output: `repr(splitter)` returns exactly `RepeatedShuffleSplit(n_repeats=3, random_state=7, test_size=0.25)`.

## Execution

| Candidate | Ran | Passed | Exit |
| --- | --- | --- | --- |
| candidate_a | True | False | 1 |
| candidate_b | True | True | 0 |

## Main artifacts

- `selected_test.patch`
- `selected_proposal.json`
- `02_execution/selected/result.json`
- `03_minimax/verdict.json`
- `pipeline.log`
