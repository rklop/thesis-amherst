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
RepeatedKFold and RepeatedStratifiedKFold do not show correct __repr__ string
#### Description

`RepeatedKFold` and `RepeatedStratifiedKFold` do not show correct \_\_repr\_\_ string.

#### Steps/Code to Reproduce

```python
>>> from sklearn.model_selection import RepeatedKFold, RepeatedStratifiedKFold
>>> repr(RepeatedKFold())
>>> repr(RepeatedStratifiedKFold())
```

#### Expected Results

```python
>>> repr(RepeatedKFold())
RepeatedKFold(n_splits=5, n_repeats=10, random_state=None)
>>> repr(RepeatedStratifiedKFold())
RepeatedStratifiedKFold(n_splits=5, n_repeats=10, random_state=None)
```

#### Actual Results

```python
>>> repr(RepeatedKFold())
'<sklearn.model_selection._split.RepeatedKFold object at 0x0000016421AA4288>'
>>> repr(RepeatedStratifiedKFold())
'<sklearn.model_selection._split.RepeatedStratifiedKFold object at 0x0000016420E115C8>'
```

#### Versions
```
System:
    python: 3.7.4 (default, Aug  9 2019, 18:34:13) [MSC v.1915 64 bit (AMD64)]
executable: D:\anaconda3\envs\xyz\python.exe
   machine: Windows-10-10.0.16299-SP0

BLAS:
    macros:
  lib_dirs:
cblas_libs: cblas

Python deps:
       pip: 19.2.2
setuptools: 41.0.1
   sklearn: 0.21.2
     numpy: 1.16.4
     scipy: 1.3.1
    Cython: None
    pandas: 0.24.2
```

</issue_statement>

<original_test_patch>
diff --git a/sklearn/model_selection/tests/test_split.py b/sklearn/model_selection/tests/test_split.py
--- a/sklearn/model_selection/tests/test_split.py
+++ b/sklearn/model_selection/tests/test_split.py
@@ -980,6 +980,17 @@ def test_repeated_cv_value_errors():
         assert_raises(ValueError, cv, n_repeats=1.5)
 
 
+@pytest.mark.parametrize(
+    "RepeatedCV", [RepeatedKFold, RepeatedStratifiedKFold]
+)
+def test_repeated_cv_repr(RepeatedCV):
+    n_splits, n_repeats = 2, 6
+    repeated_cv = RepeatedCV(n_splits=n_splits, n_repeats=n_repeats)
+    repeated_cv_repr = ('{}(n_repeats=6, n_splits=2, random_state=None)'
+                        .format(repeated_cv.__class__.__name__))
+    assert repeated_cv_repr == repr(repeated_cv)
+
+
 def test_repeated_kfold_determinstic_split():
     X = [[1, 2], [3, 4], [5, 6], [7, 8], [9, 10]]
     random_state = 258173307

</original_test_patch>

<candidate_a_patch>
diff --git a/sklearn/model_selection/_split.py b/sklearn/model_selection/_split.py
index c49a3ce6a..e98c9008e 100644
--- a/sklearn/model_selection/_split.py
+++ b/sklearn/model_selection/_split.py
@@ -1163,6 +1163,9 @@ class _RepeatedSplits(metaclass=ABCMeta):
                      **self.cvargs)
         return cv.get_n_splits(X, y, groups) * self.n_repeats
 
+    def __repr__(self):
+        return _build_repr(self)
+
 
 class RepeatedKFold(_RepeatedSplits):
     """Repeated K-Fold cross validator.
@@ -1213,6 +1216,7 @@ class RepeatedKFold(_RepeatedSplits):
     RepeatedStratifiedKFold: Repeats Stratified K-Fold n times.
     """
     def __init__(self, n_splits=5, n_repeats=10, random_state=None):
+        self.n_splits = n_splits
         super().__init__(
             KFold, n_repeats, random_state, n_splits=n_splits)
 
@@ -1266,6 +1270,7 @@ class RepeatedStratifiedKFold(_RepeatedSplits):
     RepeatedKFold: Repeats K-Fold n times.
     """
     def __init__(self, n_splits=5, n_repeats=10, random_state=None):
+        self.n_splits = n_splits
         super().__init__(
             StratifiedKFold, n_repeats, random_state, n_splits=n_splits)
 

</candidate_a_patch>

<candidate_b_patch>
diff --git a/sklearn/model_selection/_split.py b/sklearn/model_selection/_split.py
--- a/sklearn/model_selection/_split.py
+++ b/sklearn/model_selection/_split.py
@@ -1163,6 +1163,9 @@ def get_n_splits(self, X=None, y=None, groups=None):
                      **self.cvargs)
         return cv.get_n_splits(X, y, groups) * self.n_repeats
 
+    def __repr__(self):
+        return _build_repr(self)
+
 
 class RepeatedKFold(_RepeatedSplits):
     """Repeated K-Fold cross validator.
@@ -2158,6 +2161,8 @@ def _build_repr(self):
         try:
             with warnings.catch_warnings(record=True) as w:
                 value = getattr(self, key, None)
+                if value is None and hasattr(self, 'cvargs'):
+                    value = self.cvargs.get(key, None)
             if len(w) and w[0].category == DeprecationWarning:
                 # if the parameter is deprecated, don't show it
                 continue

</candidate_b_patch>

<generated_test_proposal>
{
  "test_name": "test_repeated_kfold_repr_with_read_only_n_splits_property",
  "specification_gap": "Adding repr support must not require RepeatedKFold subclasses to provide a writable n_splits instance attribute. A subclass can expose that constructor parameter through a read-only property while the base class retains the operational configuration.",
  "input_description": "Construct an RKF subclass of RepeatedKFold with a read-only n_splits property, passing n_splits=3, n_repeats=2, and random_state=0.",
  "expected_output": "Construction succeeds, get_n_splits() returns 6, and repr(splitter) returns \"RKF(n_repeats=2, n_splits=3, random_state=0)\".",
  "expected_passing_candidate": "candidate_b",
  "why_interesting": "Candidate B builds the representation without adding a new assignment requirement. Candidate A assigns self.n_splits during construction, which collides with the subclass's read-only property and raises AttributeError. This exercises real splitter behavior and subclass compatibility rather than inspecting internal fields directly.",
  "test_patch": "diff --git a/sklearn/model_selection/tests/test_split.py b/sklearn/model_selection/tests/test_split.py\n--- a/sklearn/model_selection/tests/test_split.py\n+++ b/sklearn/model_selection/tests/test_split.py\n@@ -973,6 +973,18 @@ def test_leave_one_p_group_out_error_on_fewer_number_of_groups():\n \n \n+def test_repeated_kfold_repr_with_read_only_n_splits_property():\n+    class RKF(RepeatedKFold):\n+        @property\n+        def n_splits(self):\n+            return getattr(self, 'cvargs', {}).get('n_splits')\n+\n+    splitter = RKF(n_splits=3, n_repeats=2, random_state=0)\n+    assert splitter.get_n_splits() == 6\n+    assert (repr(splitter) ==\n+            \"RKF(n_repeats=2, n_splits=3, random_state=0)\")\n+\n+\n @ignore_warnings\n def test_repeated_cv_value_errors():\n     # n_repeats is not integer or <= 0\n     for cv in (RepeatedKFold, RepeatedStratifiedKFold):\n",
  "test_command": "python -m pytest -q sklearn/model_selection/tests/test_split.py::test_repeated_kfold_repr_with_read_only_n_splits_property"
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
F                                                                        [100%]
=================================== FAILURES ===================================
__________ test_repeated_kfold_repr_with_read_only_n_splits_property ___________

    def test_repeated_kfold_repr_with_read_only_n_splits_property():
        class RKF(RepeatedKFold):
            @property
            def n_splits(self):
                return getattr(self, 'cvargs', {}).get('n_splits')
    
>       splitter = RKF(n_splits=3, n_repeats=2, random_state=0)

sklearn/model_selection/tests/test_split.py:981: 
_ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ 

self = RKF(n_repeats=None, n_splits=None, random_state=None), n_splits = 3
n_repeats = 2, random_state = 0

    def __init__(self, n_splits=5, n_repeats=10, random_state=None):
>       self.n_splits = n_splits
E       AttributeError: can't set attribute

sklearn/model_selection/_split.py:1219: AttributeError
1 failed, 1 warning in 0.55s
[pipeline] test_exit_code=1

```

## candidate_b
returncode=0 passed=True test_ran=True
log=02_execution/attempt_02/candidate_b.log
```
[pipeline] checking candidate patch
[pipeline] checking generated test patch
[pipeline] executing generated test command
.                                                                        [100%]
1 passed, 1 warning in 0.46s
[pipeline] test_exit_code=0

```
</validated_execution>
