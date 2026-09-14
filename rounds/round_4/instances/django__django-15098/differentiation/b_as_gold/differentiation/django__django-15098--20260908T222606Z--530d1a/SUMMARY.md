# Differentiating-test run: `django__django-15098`

- Status: **completed**
- Image: `docker.io/swebench/sweb.eval.x86_64.django_1776_django-15098:latest`
- Run directory: `/home/rocky/SWE-ABS/mini-swe-agent/runs/spec-refinement-core-r2-negative-filter-on-v1/round-04/bidirectional-differentiation/instances/django__django-15098/b_as_gold/differentiation/django__django-15098--20260908T222606Z--530d1a`
- Test: `test_get_language_from_path_special_fallback`
- Test command: `cd /testbed && ./tests/runtests.py i18n.tests.MiscTests.test_get_language_from_path_special_fallback`

## Specification gap

Path-based language detection must preserve LANG_INFO special fallbacks before considering generic sibling variants. Resolving a specific alias such as zh-cn must not depend on the ordering of other configured zh variants.

## Input/output contract

Input: Call get_language_from_path('/zh-cn/') with LANGUAGES containing zh, then zh-hant, then zh-hans. Django has no generic zh catalog, while zh-cn has the established special fallback zh-hans.

Expected output: The function returns 'zh-hans'. The supplied gold follows the zh-cn special fallback. The generated candidate first truncates to zh and incorrectly selects the first configured sibling, zh-hant.

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
