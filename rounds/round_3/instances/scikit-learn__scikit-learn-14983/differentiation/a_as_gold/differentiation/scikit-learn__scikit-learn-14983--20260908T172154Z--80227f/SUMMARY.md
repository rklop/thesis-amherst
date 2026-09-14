# Differentiating-test run: `scikit-learn__scikit-learn-14983`

- Status: **completed**
- Image: `docker.io/swebench/sweb.eval.x86_64.scikit-learn_1776_scikit-learn-14983:latest`
- Run directory: `/home/rocky/SWE-ABS/mini-swe-agent/runs/spec-refinement-core-r2-negative-filter-on-v1/round-03/bidirectional-differentiation/instances/scikit-learn__scikit-learn-14983/a_as_gold/differentiation/scikit-learn__scikit-learn-14983--20260908T172154Z--80227f`
- Test: `test_repeated_kfold_repr_with_write_once_parameter`
- Test command: `cd /testbed && python -m pytest -q sklearn/model_selection/tests/test_split.py::test_repeated_kfold_repr_with_write_once_parameter`

## Specification gap

RepeatedKFold must expose forwarded constructor parameters for repr without assigning the same parameter again after base initialization. This preserves support for subclasses that manage a public constructor parameter with a write-once descriptor.

## Input/output contract

Input: Instantiate a RepeatedKFold subclass whose n_splits property accepts one initialization only, with n_splits=3, n_repeats=2, and random_state=0, then call repr().

Expected output: Construction succeeds and repr returns "WriteOnceRepeatedKFold(n_repeats=2, n_splits=3, random_state=0)".

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
