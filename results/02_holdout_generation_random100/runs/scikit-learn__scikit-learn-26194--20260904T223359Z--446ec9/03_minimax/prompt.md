Judge the quality of a validated differentiating test for a
SWE-Bench task. Return exactly one JSON object and no Markdown.

Allowed ratings:

- `high_signal`: the test exercises intended public behavior, has a defensible
  oracle, reveals a meaningful missing specification, and is reasonably
  general/minimal rather than tailored to implementation internals.
- `low_signal`: the split is real but primarily reflects brittle internals,
  incidental formatting, undefined behavior, a contrived exploit, unrelated
  regressions, or an oracle not supported by the issue/API contract.
- `ambiguous`: the available evidence does not justify either quality judgment
  without a human deciding an underspecified semantic question.

Do not rate a test high merely because one candidate passes and one fails.
Assess whether the expected output follows from the issue and established API
semantics. Explicitly identify which candidate passed and whether that winner
appears more specification-conformant.

Required JSON keys: `rating`, `confidence`, `summary`, `specification_signal`,
`oracle_quality`, `concerns`, `human_review_questions`. `rating` must be one of
the three allowed values. `confidence` must be a number from 0 to 1. The last
three keys may contain arrays of strings.

<issue_statement>
Thresholds can exceed 1 in `roc_curve` while providing probability estimate
While working on https://github.com/scikit-learn/scikit-learn/pull/26120, I found out that something was odd with `roc_curve` that returns a threshold greater than 1. A non-regression test (that could be part of `sklearn/metrics/tests/test_ranking.py`) could be as follow:

```python
def test_roc_curve_with_probablity_estimates():
    rng = np.random.RandomState(42)
    y_true = rng.randint(0, 2, size=10)
    y_score = rng.rand(10)
    _, _, thresholds = roc_curve(y_true, y_score)
    assert np.logical_or(thresholds <= 1, thresholds >= 0).all()
```

The reason is due to the following:

https://github.com/scikit-learn/scikit-learn/blob/e886ce4e1444c61b865e7839c9cff5464ee20ace/sklearn/metrics/_ranking.py#L1086

Basically, this is to add a point for `fpr=0` and `tpr=0`. However, the `+ 1` rule does not make sense in the case `y_score` is a probability estimate.

I am not sure what would be the best fix here. A potential workaround would be to check `thresholds.max() <= 1` in which case we should clip `thresholds` to not be above 1.

</issue_statement>

<original_test_patch>
diff --git a/sklearn/metrics/tests/test_ranking.py b/sklearn/metrics/tests/test_ranking.py
--- a/sklearn/metrics/tests/test_ranking.py
+++ b/sklearn/metrics/tests/test_ranking.py
@@ -418,13 +418,13 @@ def test_roc_curve_drop_intermediate():
     y_true = [0, 0, 0, 0, 1, 1]
     y_score = [0.0, 0.2, 0.5, 0.6, 0.7, 1.0]
     tpr, fpr, thresholds = roc_curve(y_true, y_score, drop_intermediate=True)
-    assert_array_almost_equal(thresholds, [2.0, 1.0, 0.7, 0.0])
+    assert_array_almost_equal(thresholds, [np.inf, 1.0, 0.7, 0.0])
 
     # Test dropping thresholds with repeating scores
     y_true = [0, 0, 0, 0, 0, 0, 0, 1, 1, 1, 1, 1, 1]
     y_score = [0.0, 0.1, 0.6, 0.6, 0.7, 0.8, 0.9, 0.6, 0.7, 0.8, 0.9, 0.9, 1.0]
     tpr, fpr, thresholds = roc_curve(y_true, y_score, drop_intermediate=True)
-    assert_array_almost_equal(thresholds, [2.0, 1.0, 0.9, 0.7, 0.6, 0.0])
+    assert_array_almost_equal(thresholds, [np.inf, 1.0, 0.9, 0.7, 0.6, 0.0])
 
 
 def test_roc_curve_fpr_tpr_increasing():
@@ -2199,3 +2199,17 @@ def test_ranking_metric_pos_label_types(metric, classes):
         assert not np.isnan(metric_1).any()
         assert not np.isnan(metric_2).any()
         assert not np.isnan(thresholds).any()
+
+
+def test_roc_curve_with_probablity_estimates(global_random_seed):
+    """Check that thresholds do not exceed 1.0 when `y_score` is a probability
+    estimate.
+
+    Non-regression test for:
+    https://github.com/scikit-learn/scikit-learn/issues/26193
+    """
+    rng = np.random.RandomState(global_random_seed)
+    y_true = rng.randint(0, 2, size=10)
+    y_score = rng.rand(10)
+    _, _, thresholds = roc_curve(y_true, y_score)
+    assert np.isinf(thresholds[0])

</original_test_patch>

<candidate_a_patch>
diff --git a/sklearn/metrics/_ranking.py b/sklearn/metrics/_ranking.py
index 0cdead923..7857a1484 100644
--- a/sklearn/metrics/_ranking.py
+++ b/sklearn/metrics/_ranking.py
@@ -1083,7 +1083,7 @@ def roc_curve(
     # to make sure that the curve starts at (0, 0)
     tps = np.r_[0, tps]
     fps = np.r_[0, fps]
-    thresholds = np.r_[thresholds[0] + 1, thresholds]
+    thresholds = np.r_[np.inf, thresholds]
 
     if fps[-1] <= 0:
         warnings.warn(

</candidate_a_patch>

<candidate_b_patch>
diff --git a/sklearn/metrics/_ranking.py b/sklearn/metrics/_ranking.py
--- a/sklearn/metrics/_ranking.py
+++ b/sklearn/metrics/_ranking.py
@@ -1016,10 +1016,10 @@ def roc_curve(
         Increasing true positive rates such that element `i` is the true
         positive rate of predictions with score >= `thresholds[i]`.
 
-    thresholds : ndarray of shape = (n_thresholds,)
+    thresholds : ndarray of shape (n_thresholds,)
         Decreasing thresholds on the decision function used to compute
         fpr and tpr. `thresholds[0]` represents no instances being predicted
-        and is arbitrarily set to `max(y_score) + 1`.
+        and is arbitrarily set to `np.inf`.
 
     See Also
     --------
@@ -1036,6 +1036,10 @@ def roc_curve(
     are reversed upon returning them to ensure they correspond to both ``fpr``
     and ``tpr``, which are sorted in reversed order during their calculation.
 
+    An arbritrary threshold is added for the case `tpr=0` and `fpr=0` to
+    ensure that the curve starts at `(0, 0)`. This threshold corresponds to the
+    `np.inf`.
+
     References
     ----------
     .. [1] `Wikipedia entry for the Receiver operating characteristic
@@ -1056,7 +1060,7 @@ def roc_curve(
     >>> tpr
     array([0. , 0.5, 0.5, 1. , 1. ])
     >>> thresholds
-    array([1.8 , 0.8 , 0.4 , 0.35, 0.1 ])
+    array([ inf, 0.8 , 0.4 , 0.35, 0.1 ])
     """
     fps, tps, thresholds = _binary_clf_curve(
         y_true, y_score, pos_label=pos_label, sample_weight=sample_weight
@@ -1083,7 +1087,8 @@ def roc_curve(
     # to make sure that the curve starts at (0, 0)
     tps = np.r_[0, tps]
     fps = np.r_[0, fps]
-    thresholds = np.r_[thresholds[0] + 1, thresholds]
+    # get dtype of `y_score` even if it is an array-like
+    thresholds = np.r_[np.inf, thresholds]
 
     if fps[-1] <= 0:
         warnings.warn(

</candidate_b_patch>

<generated_test_proposal>
{
  "test_name": "roc_curve_public_example_consistency",
  "specification_gap": "The implementation change affects a public sentinel value, so the documented return contract and executable API example must change with it. Candidate A returns infinity but still publicly promises and demonstrates `max(y_score) + 1`; candidate B makes the runtime behavior, return documentation, and example consistent.",
  "input_description": "Execute only the `roc_curve` public docstring examples, including `y = [1, 1, 2, 2]`, scores `[0.1, 0.4, 0.35, 0.8]`, and `pos_label=2`.",
  "expected_output": "The example succeeds and displays thresholds as `array([ inf, 0.8 , 0.4 , 0.35, 0.1 ])`, with the leading infinity representing the no-predicted-instances point. Candidate A instead documents `1.8` while returning `inf`, causing one doctest failure.",
  "expected_passing_candidate": "candidate_b",
  "why_interesting": "This exercises the public function through its executable example and enforces the general invariant that an API behavior change must not leave its published input/output contract stale. It distinguishes the candidates without relying on private implementation details.",
  "test_patch": "diff --git a/sklearn/metrics/tests/test_ranking.py b/sklearn/metrics/tests/test_ranking.py\n--- a/sklearn/metrics/tests/test_ranking.py\n+++ b/sklearn/metrics/tests/test_ranking.py\n@@ -1,3 +1,4 @@\n+import doctest\n import re\n import pytest\n import numpy as np\n@@ -200,9 +201,15 @@ def test_roc_curve(drop):\n     assert_array_almost_equal(roc_auc, expected_auc, decimal=2)\n     assert_almost_equal(roc_auc, roc_auc_score(y_true, y_score))\n     assert fpr.shape == tpr.shape\n     assert fpr.shape == thresholds.shape\n \n \n+def test_roc_curve_docstring_example():\n+    [doc_test] = doctest.DocTestFinder().find(roc_curve)\n+    result = doctest.DocTestRunner().run(doc_test)\n+    assert result.failed == 0\n+\n+\n def test_roc_curve_end_points():\n     # Make sure that roc_curve returns a curve start at 0 and ending and\n     # 1 even in corner cases\n",
  "test_command": "python -m pytest -q sklearn/metrics/tests/test_ranking.py::test_roc_curve_docstring_example"
}
</generated_test_proposal>

<validated_execution>
## candidate_a
returncode=1 passed=False test_ran=True
log=02_execution/attempt_02/candidate_a.log
```
[pipeline] checking candidate patch
[pipeline] checking generated test patch
[pipeline] executing generated test command
[31mF[0m[31m                                                                        [100%][0m
=================================== FAILURES ===================================
[31m[1m_______________________ test_roc_curve_docstring_example _______________________[0m

    def test_roc_curve_docstring_example():
        [doc_test] = doctest.DocTestFinder().find(roc_curve)
        result = doctest.DocTestRunner().run(doc_test)
>       assert result.failed == 0
[1m[31mE       assert 1 == 0[0m
[1m[31mE        +  where 1 = TestResults(failed=1, attempted=8).failed[0m

[1m[31msklearn/metrics/tests/test_ranking.py[0m:210: AssertionError
----------------------------- Captured stdout call -----------------------------
**********************************************************************
File "/testbed/sklearn/metrics/_ranking.py", line 280, in roc_curve
Failed example:
    thresholds
Expected:
    array([1.8 , 0.8 , 0.4 , 0.35, 0.1 ])
Got:
    array([ inf, 0.8 , 0.4 , 0.35, 0.1 ])
[31m[31m[1m1 failed[0m[31m in 0.39s[0m[0m
[pipeline] test_exit_code=1

```

## candidate_b
returncode=0 passed=True test_ran=True
log=02_execution/attempt_02/candidate_b.log
```
[pipeline] checking candidate patch
[pipeline] checking generated test patch
[pipeline] executing generated test command
[32m.[0m[32m                                                                        [100%][0m
[32m[32m[1m1 passed[0m[32m in 0.27s[0m[0m
[pipeline] test_exit_code=0

```
</validated_execution>
