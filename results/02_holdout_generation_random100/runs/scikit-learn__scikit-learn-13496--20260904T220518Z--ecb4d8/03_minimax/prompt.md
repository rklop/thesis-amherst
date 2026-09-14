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
Expose warm_start in Isolation forest
It seems to me that `sklearn.ensemble.IsolationForest` supports incremental addition of new trees with the `warm_start` parameter of its parent class, `sklearn.ensemble.BaseBagging`.

Even though this parameter is not exposed in `__init__()` , it gets inherited from `BaseBagging` and one can use it by changing it to `True` after initialization. To make it work, you have to also increment `n_estimators` on every iteration. 

It took me a while to notice that it actually works, and I had to inspect the source code of both `IsolationForest` and `BaseBagging`. Also, it looks to me that the behavior is in-line with `sklearn.ensemble.BaseForest` that is behind e.g. `sklearn.ensemble.RandomForestClassifier`.

To make it more easier to use, I'd suggest to:
* expose `warm_start` in `IsolationForest.__init__()`, default `False`;
* document it in the same way as it is documented for `RandomForestClassifier`, i.e. say:
```py
    warm_start : bool, optional (default=False)
        When set to ``True``, reuse the solution of the previous call to fit
        and add more estimators to the ensemble, otherwise, just fit a whole
        new forest. See :term:`the Glossary <warm_start>`.
```
* add a test to make sure it works properly;
* possibly also mention in the "IsolationForest example" documentation entry;


</issue_statement>

<original_test_patch>
diff --git a/sklearn/ensemble/tests/test_iforest.py b/sklearn/ensemble/tests/test_iforest.py
--- a/sklearn/ensemble/tests/test_iforest.py
+++ b/sklearn/ensemble/tests/test_iforest.py
@@ -295,6 +295,28 @@ def test_score_samples():
                        clf2.score_samples([[2., 2.]]))
 
 
+@pytest.mark.filterwarnings('ignore:default contamination')
+@pytest.mark.filterwarnings('ignore:behaviour="old"')
+def test_iforest_warm_start():
+    """Test iterative addition of iTrees to an iForest """
+
+    rng = check_random_state(0)
+    X = rng.randn(20, 2)
+
+    # fit first 10 trees
+    clf = IsolationForest(n_estimators=10, max_samples=20,
+                          random_state=rng, warm_start=True)
+    clf.fit(X)
+    # remember the 1st tree
+    tree_1 = clf.estimators_[0]
+    # fit another 10 trees
+    clf.set_params(n_estimators=20)
+    clf.fit(X)
+    # expecting 20 fitted trees and no overwritten trees
+    assert len(clf.estimators_) == 20
+    assert clf.estimators_[0] is tree_1
+
+
 @pytest.mark.filterwarnings('ignore:default contamination')
 @pytest.mark.filterwarnings('ignore:behaviour="old"')
 def test_deprecation():

</original_test_patch>

<candidate_a_patch>
diff --git a/sklearn/ensemble/iforest.py b/sklearn/ensemble/iforest.py
index 8a1bd3625..589171f73 100644
--- a/sklearn/ensemble/iforest.py
+++ b/sklearn/ensemble/iforest.py
@@ -120,6 +120,10 @@ class IsolationForest(BaseBagging, OutlierMixin):
     verbose : int, optional (default=0)
         Controls the verbosity of the tree building process.
 
+    warm_start : bool, optional (default=False)
+        When set to ``True``, reuse the solution of the previous call to fit
+        and add more estimators to the ensemble, otherwise, just fit a whole
+        new forest. See :term:`the Glossary <warm_start>`.
 
     Attributes
     ----------
@@ -173,7 +177,8 @@ class IsolationForest(BaseBagging, OutlierMixin):
                  n_jobs=None,
                  behaviour='old',
                  random_state=None,
-                 verbose=0):
+                 verbose=0,
+                 warm_start=False):
         super().__init__(
             base_estimator=ExtraTreeRegressor(
                 max_features=1,
@@ -185,6 +190,7 @@ class IsolationForest(BaseBagging, OutlierMixin):
             n_estimators=n_estimators,
             max_samples=max_samples,
             max_features=max_features,
+            warm_start=warm_start,
             n_jobs=n_jobs,
             random_state=random_state,
             verbose=verbose)

</candidate_a_patch>

<candidate_b_patch>
diff --git a/sklearn/ensemble/iforest.py b/sklearn/ensemble/iforest.py
--- a/sklearn/ensemble/iforest.py
+++ b/sklearn/ensemble/iforest.py
@@ -120,6 +120,12 @@ class IsolationForest(BaseBagging, OutlierMixin):
     verbose : int, optional (default=0)
         Controls the verbosity of the tree building process.
 
+    warm_start : bool, optional (default=False)
+        When set to ``True``, reuse the solution of the previous call to fit
+        and add more estimators to the ensemble, otherwise, just fit a whole
+        new forest. See :term:`the Glossary <warm_start>`.
+
+        .. versionadded:: 0.21
 
     Attributes
     ----------
@@ -173,7 +179,8 @@ def __init__(self,
                  n_jobs=None,
                  behaviour='old',
                  random_state=None,
-                 verbose=0):
+                 verbose=0,
+                 warm_start=False):
         super().__init__(
             base_estimator=ExtraTreeRegressor(
                 max_features=1,
@@ -185,6 +192,7 @@ def __init__(self,
             n_estimators=n_estimators,
             max_samples=max_samples,
             max_features=max_features,
+            warm_start=warm_start,
             n_jobs=n_jobs,
             random_state=random_state,
             verbose=verbose)

</candidate_b_patch>

<generated_test_proposal>
{
  "test_name": "test_warm_start_documented_as_new_in_0_21",
  "specification_gap": "The newly exposed public constructor parameter should identify the release in which it became part of IsolationForest's constructor API. This distinguishes it from the previously inherited, undiscoverable warm_start attribute.",
  "input_description": "Inspect the public IsolationForest class docstring, isolate the warm_start parameter section, and query its version-introduction metadata.",
  "expected_output": "The warm_start parameter section contains the semantic reStructuredText directive '.. versionadded:: 0.21', rendered in generated documentation as 'New in version 0.21.'",
  "expected_passing_candidate": "candidate_b",
  "why_interesting": "Both candidates implement identical runtime behavior, so fitting and estimator-parameter tests cannot distinguish them. Candidate B additionally records the public API provenance of the newly exposed constructor parameter; candidate A omits it.",
  "test_patch": "diff --git a/sklearn/ensemble/tests/test_iforest.py b/sklearn/ensemble/tests/test_iforest.py\n--- a/sklearn/ensemble/tests/test_iforest.py\n+++ b/sklearn/ensemble/tests/test_iforest.py\n@@ -46,6 +46,13 @@\n perm = rng.permutation(boston.target.size)\n boston.data = boston.data[perm]\n boston.target = boston.target[perm]\n \n \n+def test_warm_start_documented_as_new_in_0_21():\n+    doc = IsolationForest.__doc__\n+    warm_start_doc = doc.split(\"warm_start : bool\", 1)[1]\n+    warm_start_doc = warm_start_doc.split(\"Attributes\", 1)[0]\n+    assert \".. versionadded:: 0.21\" in warm_start_doc\n+\n+\n @pytest.mark.filterwarnings('ignore:threshold_ attribute')\n def test_iforest():\n",
  "test_command": "pytest -q sklearn/ensemble/tests/test_iforest.py::test_warm_start_documented_as_new_in_0_21"
}
</generated_test_proposal>

<validated_execution>
## candidate_a
returncode=1 passed=False test_ran=True
log=02_execution/attempt_03/candidate_a.log
```
[pipeline] checking candidate patch
[pipeline] checking generated test patch
[pipeline] executing generated test command
F                                                                        [100%]
=================================== FAILURES ===================================
__________________ test_warm_start_documented_as_new_in_0_21 ___________________

    def test_warm_start_documented_as_new_in_0_21():
        doc = IsolationForest.__doc__
        warm_start_doc = doc.split("warm_start : bool", 1)[1]
        warm_start_doc = warm_start_doc.split("Attributes", 1)[0]
>       assert ".. versionadded:: 0.21" in warm_start_doc
E       AssertionError: assert '.. versionadded:: 0.21' in ', optional (default=False)\n        When set to ``True``, reuse the solution of the previous call to fit\n        and...ators to the ensemble, otherwise, just fit a whole\n        new forest. See :term:`the Glossary <warm_start>`.\n\n    '

sklearn/ensemble/tests/test_iforest.py:55: AssertionError
1 failed in 0.75s
[pipeline] test_exit_code=1

```

## candidate_b
returncode=0 passed=True test_ran=True
log=02_execution/attempt_03/candidate_b.log
```
[pipeline] checking candidate patch
[pipeline] checking generated test patch
[pipeline] executing generated test command
.                                                                        [100%]
1 passed in 0.69s
[pipeline] test_exit_code=0

```
</validated_execution>
