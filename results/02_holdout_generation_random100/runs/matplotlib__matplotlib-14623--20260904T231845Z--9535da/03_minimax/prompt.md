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
Inverting an axis using its limits does not work for log scale
### Bug report

**Bug summary**
Starting in matplotlib 3.1.0 it is no longer possible to invert a log axis using its limits.

**Code for reproduction**
```python
import numpy as np
import matplotlib.pyplot as plt


y = np.linspace(1000e2, 1, 100)
x = np.exp(-np.linspace(0, 1, y.size))

for yscale in ('linear', 'log'):
    fig, ax = plt.subplots()
    ax.plot(x, y)
    ax.set_yscale(yscale)
    ax.set_ylim(y.max(), y.min())
```

**Actual outcome**
The yaxis is only inverted for the ``"linear"`` scale.

![linear](https://user-images.githubusercontent.com/9482218/60081191-99245e80-9731-11e9-9e4a-eadb3ef58666.png)

![log](https://user-images.githubusercontent.com/9482218/60081203-9e81a900-9731-11e9-8bae-0be1c9762b16.png)

**Expected outcome**
I would expect the yaxis to be inverted for both the ``"linear"`` and the ``"log"`` scale.

**Matplotlib version**
  * Operating system: Linux and MacOS
  * Matplotlib version: 3.1.0 
  * Python version: 3.7.3
 
Python and matplotlib have been installed using conda.


</issue_statement>

<original_test_patch>
diff --git a/lib/matplotlib/tests/test_axes.py b/lib/matplotlib/tests/test_axes.py
--- a/lib/matplotlib/tests/test_axes.py
+++ b/lib/matplotlib/tests/test_axes.py
@@ -936,7 +936,12 @@ def test_inverted_limits():
 
     assert ax.get_xlim() == (-5, 4)
     assert ax.get_ylim() == (5, -3)
-    plt.close()
+
+    # Test inverting nonlinear axes.
+    fig, ax = plt.subplots()
+    ax.set_yscale("log")
+    ax.set_ylim(10, 1)
+    assert ax.get_ylim() == (10, 1)
 
 
 @image_comparison(baseline_images=['nonfinite_limits'])

</original_test_patch>

<candidate_a_patch>
diff --git a/lib/matplotlib/ticker.py b/lib/matplotlib/ticker.py
index 209a3f92c..e5a2e13da 100644
--- a/lib/matplotlib/ticker.py
+++ b/lib/matplotlib/ticker.py
@@ -2380,7 +2380,10 @@ class LogLocator(Locator):
             return 1, 10  # initial range, no data plotted yet
 
         if vmin > vmax:
-            vmin, vmax = vmax, vmin
+            # If vmax > 0, the user is inverting the axis intentionally.
+            # Only swap if vmax <= 0 (which would be invalid for log scale).
+            if vmax <= 0:
+                vmin, vmax = vmax, vmin
         if vmax <= 0:
             cbook._warn_external(
                 "Data has no positive values, and therefore cannot be "

</candidate_a_patch>

<candidate_b_patch>
diff --git a/lib/matplotlib/axes/_base.py b/lib/matplotlib/axes/_base.py
--- a/lib/matplotlib/axes/_base.py
+++ b/lib/matplotlib/axes/_base.py
@@ -3262,8 +3262,11 @@ def set_xlim(self, left=None, right=None, emit=True, auto=False,
             cbook._warn_external(
                 f"Attempting to set identical left == right == {left} results "
                 f"in singular transformations; automatically expanding.")
+        swapped = left > right
         left, right = self.xaxis.get_major_locator().nonsingular(left, right)
         left, right = self.xaxis.limit_range_for_scale(left, right)
+        if swapped:
+            left, right = right, left
 
         self.viewLim.intervalx = (left, right)
         if auto is not None:
@@ -3642,8 +3645,11 @@ def set_ylim(self, bottom=None, top=None, emit=True, auto=False,
                 f"Attempting to set identical bottom == top == {bottom} "
                 f"results in singular transformations; automatically "
                 f"expanding.")
+        swapped = bottom > top
         bottom, top = self.yaxis.get_major_locator().nonsingular(bottom, top)
         bottom, top = self.yaxis.limit_range_for_scale(bottom, top)
+        if swapped:
+            bottom, top = top, bottom
 
         self.viewLim.intervaly = (bottom, top)
         if auto is not None:
diff --git a/lib/matplotlib/ticker.py b/lib/matplotlib/ticker.py
--- a/lib/matplotlib/ticker.py
+++ b/lib/matplotlib/ticker.py
@@ -1521,8 +1521,8 @@ def raise_if_exceeds(self, locs):
         return locs
 
     def nonsingular(self, v0, v1):
-        """Modify the endpoints of a range as needed to avoid singularities."""
-        return mtransforms.nonsingular(v0, v1, increasing=False, expander=.05)
+        """Expand a range as needed to avoid singularities."""
+        return mtransforms.nonsingular(v0, v1, expander=.05)
 
     def view_limits(self, vmin, vmax):
         """
diff --git a/lib/mpl_toolkits/mplot3d/axes3d.py b/lib/mpl_toolkits/mplot3d/axes3d.py
--- a/lib/mpl_toolkits/mplot3d/axes3d.py
+++ b/lib/mpl_toolkits/mplot3d/axes3d.py
@@ -623,8 +623,11 @@ def set_xlim3d(self, left=None, right=None, emit=True, auto=False,
             cbook._warn_external(
                 f"Attempting to set identical left == right == {left} results "
                 f"in singular transformations; automatically expanding.")
+        swapped = left > right
         left, right = self.xaxis.get_major_locator().nonsingular(left, right)
         left, right = self.xaxis.limit_range_for_scale(left, right)
+        if swapped:
+            left, right = right, left
         self.xy_viewLim.intervalx = (left, right)
 
         if auto is not None:
@@ -681,8 +684,11 @@ def set_ylim3d(self, bottom=None, top=None, emit=True, auto=False,
                 f"Attempting to set identical bottom == top == {bottom} "
                 f"results in singular transformations; automatically "
                 f"expanding.")
+        swapped = bottom > top
         bottom, top = self.yaxis.get_major_locator().nonsingular(bottom, top)
         bottom, top = self.yaxis.limit_range_for_scale(bottom, top)
+        if swapped:
+            bottom, top = top, bottom
         self.xy_viewLim.intervaly = (bottom, top)
 
         if auto is not None:
@@ -739,8 +745,11 @@ def set_zlim3d(self, bottom=None, top=None, emit=True, auto=False,
                 f"Attempting to set identical bottom == top == {bottom} "
                 f"results in singular transformations; automatically "
                 f"expanding.")
+        swapped = bottom > top
         bottom, top = self.zaxis.get_major_locator().nonsingular(bottom, top)
         bottom, top = self.zaxis.limit_range_for_scale(bottom, top)
+        if swapped:
+            bottom, top = top, bottom
         self.zz_viewLim.intervalx = (bottom, top)
 
         if auto is not None:

</candidate_b_patch>

<generated_test_proposal>
{
  "test_name": "test_nonsingular_reversed_limits",
  "specification_gap": "Axis inversion must be preserved by Axes.set_*lim without changing LogLocator.nonsingular's public normalization behavior. For distinct positive limits, the locator method should return increasing bounds.",
  "input_description": "Attach a LogLocator to a logarithmic x-axis and call nonsingular(1000, 1) with reversed positive finite bounds.",
  "expected_output": "The public method returns (1, 1000). Candidate A instead returns (1000, 1), while candidate B retains normalization and preserves inversion at the Axes layer.",
  "expected_passing_candidate": "candidate_b",
  "why_interesting": "This exercises an alternate public entry point and distinguishes locator range normalization from visual axis direction. It avoids rendering, private state, and implementation-detail assertions.",
  "test_patch": "diff --git a/lib/matplotlib/tests/test_ticker.py b/lib/matplotlib/tests/test_ticker.py\n--- a/lib/matplotlib/tests/test_ticker.py\n+++ b/lib/matplotlib/tests/test_ticker.py\n@@ -225,6 +225,11 @@\n         assert loc._base == 4\n         assert list(loc._subs) == [2.0]\n \n+    def test_nonsingular_reversed_limits(self):\n+        fig, ax = plt.subplots()\n+        ax.set_xscale('log')\n+        assert ax.xaxis.get_major_locator().nonsingular(1000, 1) == (1, 1000)\n+\n \n class TestNullLocator(object):\n     def test_set_params(self):\n",
  "test_command": "python -m pytest -q lib/matplotlib/tests/test_ticker.py::TestLogLocator::test_nonsingular_reversed_limits"
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
Matplotlib is not built with the correct FreeType version to run tests.  Set local_freetype=True in setup.cfg and rebuild. Expect many image comparison failures below. Expected freetype version 2.6.1. Found freetype version 2.11.1. Freetype build type is not local
F                                                                        [100%]
=================================== FAILURES ===================================
_______________ TestLogLocator.test_nonsingular_reversed_limits ________________

self = <matplotlib.tests.test_ticker.TestLogLocator object at 0x7665038183a0>

    def test_nonsingular_reversed_limits(self):
        fig, ax = plt.subplots()
        ax.set_xscale('log')
>       assert ax.xaxis.get_major_locator().nonsingular(1000, 1) == (1, 1000)
E       assert (1000, 1) == (1, 1000)
E         
E         At index 0 diff: 1000 != 1
E         Use -v to get more diff

lib/matplotlib/tests/test_ticker.py:231: AssertionError
=============================== warnings summary ===============================
lib/matplotlib/__init__.py:200
lib/matplotlib/__init__.py:200
lib/matplotlib/__init__.py:200
lib/matplotlib/__init__.py:200
lib/matplotlib/__init__.py:200
  /testbed/lib/matplotlib/__init__.py:200: DeprecationWarning: distutils Version classes are deprecated. Use packaging.version instead.
    if LooseVersion(module.__version__) < minver:

../opt/miniconda3/envs/testbed/lib/python3.8/site-packages/setuptools/_distutils/version.py:337
../opt/miniconda3/envs/testbed/lib/python3.8/site-packages/setuptools/_distutils/version.py:337
../opt/miniconda3/envs/testbed/lib/python3.8/site-packages/setuptools/_distutils/version.py:337
../opt/miniconda3/envs/testbed/lib/python3.8/site-packages/setuptools/_distutils/version.py:337
../opt/miniconda3/envs/testbed/lib/python3.8/site-packages/setuptools/_distutils/version.py:337
lib/matplotlib/tests/test_ticker.py::TestLogLocator::test_nonsingular_reversed_limits
  /opt/miniconda3/envs/testbed/lib/python3.8/site-packages/setuptools/_distutils/version.py:337: DeprecationWarning: distutils Version classes are deprecated. Use packaging.version instead.
    other = LooseVersion(other)

lib/matplotlib/tests/test_ticker.py::TestLogLocator::test_nonsingular_reversed_limits
  /testbed/lib/matplotlib/__init__.py:332: DeprecationWarning: distutils Version classes are deprecated. Use packaging.version instead.
    version = LooseVersion(match.group(1))

-- Docs: https://docs.pytest.org/en/stable/how-to/capture-warnings.html
=========================== short test summary info ============================
FAILED lib/matplotlib/tests/test_ticker.py::TestLogLocator::test_nonsingular_reversed_limits
1 failed, 12 warnings in 2.10s
[pipeline] test_exit_code=1

```

## candidate_b
returncode=0 passed=True test_ran=True
log=02_execution/attempt_02/candidate_b.log
```
[pipeline] checking candidate patch
[pipeline] checking generated test patch
[pipeline] executing generated test command
Matplotlib is not built with the correct FreeType version to run tests.  Set local_freetype=True in setup.cfg and rebuild. Expect many image comparison failures below. Expected freetype version 2.6.1. Found freetype version 2.11.1. Freetype build type is not local
.                                                                        [100%]
=============================== warnings summary ===============================
lib/matplotlib/__init__.py:200
lib/matplotlib/__init__.py:200
lib/matplotlib/__init__.py:200
lib/matplotlib/__init__.py:200
lib/matplotlib/__init__.py:200
  /testbed/lib/matplotlib/__init__.py:200: DeprecationWarning: distutils Version classes are deprecated. Use packaging.version instead.
    if LooseVersion(module.__version__) < minver:

../opt/miniconda3/envs/testbed/lib/python3.8/site-packages/setuptools/_distutils/version.py:337
../opt/miniconda3/envs/testbed/lib/python3.8/site-packages/setuptools/_distutils/version.py:337
../opt/miniconda3/envs/testbed/lib/python3.8/site-packages/setuptools/_distutils/version.py:337
../opt/miniconda3/envs/testbed/lib/python3.8/site-packages/setuptools/_distutils/version.py:337
../opt/miniconda3/envs/testbed/lib/python3.8/site-packages/setuptools/_distutils/version.py:337
lib/matplotlib/tests/test_ticker.py::TestLogLocator::test_nonsingular_reversed_limits
  /opt/miniconda3/envs/testbed/lib/python3.8/site-packages/setuptools/_distutils/version.py:337: DeprecationWarning: distutils Version classes are deprecated. Use packaging.version instead.
    other = LooseVersion(other)

lib/matplotlib/tests/test_ticker.py::TestLogLocator::test_nonsingular_reversed_limits
  /testbed/lib/matplotlib/__init__.py:332: DeprecationWarning: distutils Version classes are deprecated. Use packaging.version instead.
    version = LooseVersion(match.group(1))

-- Docs: https://docs.pytest.org/en/stable/how-to/capture-warnings.html
1 passed, 12 warnings in 1.92s
[pipeline] test_exit_code=0

```
</validated_execution>
