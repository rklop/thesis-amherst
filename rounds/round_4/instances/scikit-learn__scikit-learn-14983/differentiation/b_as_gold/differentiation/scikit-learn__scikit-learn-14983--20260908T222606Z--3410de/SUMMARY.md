# Differentiating-test run: `scikit-learn__scikit-learn-14983`

- Status: **completed**
- Image: `docker.io/swebench/sweb.eval.x86_64.scikit-learn_1776_scikit-learn-14983:latest`
- Run directory: `/home/rocky/SWE-ABS/mini-swe-agent/runs/spec-refinement-core-r2-negative-filter-on-v1/round-04/bidirectional-differentiation/instances/scikit-learn__scikit-learn-14983/b_as_gold/differentiation/scikit-learn__scikit-learn-14983--20260908T222606Z--3410de`
- Test: `test_repeated_split_repr_ignores_deprecated_parameters`
- Test command: `cd /testbed && python -m pytest -q sklearn/model_selection/tests/test_split.py::test_repeated_split_repr_ignores_deprecated_parameters`

## Specification gap

Repeated cross-validator representations should preserve the established `_build_repr` behavior for inherited public subclasses: deprecated constructor parameters are omitted rather than exposed. The generated candidate reimplements repr and includes them.

## Input/output contract

Input: Construct a `RepeatedKFold` subclass with `n_splits=3` and a deprecated `legacy='ignored'` constructor property, then call `repr`.

Expected output: The observable representation is exactly `RepeatedKFoldWithDeprecatedParameter(n_splits=3)`, with the deprecated `legacy` parameter omitted.

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
