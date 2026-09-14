# Differentiating-test run: `scikit-learn__scikit-learn-14983`

- Status: **completed**
- Image: `docker.io/swebench/sweb.eval.x86_64.scikit-learn_1776_scikit-learn-14983:latest`
- Run directory: `/home/rocky/SWE-ABS/mini-swe-agent/runs/spec-refinement-core-r2-negative-filter-on-v1/round-03/bidirectional-differentiation/instances/scikit-learn__scikit-learn-14983/b_as_gold/differentiation/scikit-learn__scikit-learn-14983--20260908T172154Z--163ab7`
- Test: `test_repeated_cv_concrete_n_splits_assignment`
- Test command: `cd /testbed && python -m pytest -q sklearn/model_selection/tests/test_split.py::test_repeated_cv_concrete_n_splits_assignment`

## Specification gap

The concrete public repeated-fold constructors retain their declared `n_splits` parameter after delegating to the shared repeated-split initializer. Thus normal subclass attribute hooks observe both forwarding and the concrete assignment; relying only on generic `**cvargs` promotion is not equivalent.

## Input/output contract

Input: Construct subclasses of `RepeatedKFold` and `RepeatedStratifiedKFold` with `n_splits=3`, `n_repeats=2`, and `random_state=11`. Each subclass records writes to the public `n_splits` attribute.

Expected output: For each class, the assignment trace is `[3, 3]`, the resulting `n_splits` attribute is `3`, and repr returns `RecordingRepeatedCV(n_repeats=2, n_splits=3, random_state=11)`.

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
