Act as a careful human software engineer reviewing a potential
negative specification test. Return exactly one JSON object and no Markdown.

The generated test separates two plausible patches, but the actual SWE-bench
gold patch executes and fails it. The question is not whether the assertion
should pass. The question is whether this failure reliably specifies behavior
that a final patch must NOT be required to implement. Extract only a narrow,
implementation-independent negative constraint supported by the issue and gold
evidence. Do not infer that every gold failure is useful.

Use this policy:
- `accept` only when the target test demonstrably ran, its failure is behavioral
  (not collection/setup/infrastructure), and the assertion captures a plausible,
  issue-relevant behavior that the gold evidence specifically declines to require.
- `reject` for infrastructure or collection failures, incidental output, private
  implementation coupling, unrelated behavior, unstable/undefined behavior, or
  when failure does not justify a reusable negative constraint.
- `human_review` only for a consequential ambiguity not resolvable from evidence.

eligible_failure_fingerprint=true. If false, do not accept.
An arbitrary nonzero exit is never sufficient.

Required JSON keys: `decision`, `confidence`, `negative_specification`,
`forbidden_behavior`, `gold_failure_interpretation`, `specification_value`,
`decision_rationale`, `concerns`, `human_review_questions`.

<issue_statement>

</issue_statement>
<candidate_a_patch>
diff --git a/lib/matplotlib/ticker.py b/lib/matplotlib/ticker.py
index 209a3f92c..22d0db12d 100644
--- a/lib/matplotlib/ticker.py
+++ b/lib/matplotlib/ticker.py
@@ -2379,22 +2379,41 @@ class LogLocator(Locator):
         if not np.isfinite(vmin) or not np.isfinite(vmax):
             return 1, 10  # initial range, no data plotted yet
 
-        if vmin > vmax:
-            vmin, vmax = vmax, vmin
-        if vmax <= 0:
-            cbook._warn_external(
-                "Data has no positive values, and therefore cannot be "
-                "log-scaled.")
-            return 1, 10
+        # If user provides inverted limits (vmin > vmax), preserve the
+        # inversion after processing if both endpoints are positive.
+        preserve_inversion = vmin > vmax and vmin > 0 and vmax > 0
 
         minpos = self.axis.get_minpos()
         if not np.isfinite(minpos):
             minpos = 1e-300  # This should never take effect.
+
+        if vmax <= 0:
+            if vmin > 0:
+                # Only vmax is non-positive, replace with minpos
+                vmax = minpos
+                # Don't preserve inversion since vmax was replaced
+                preserve_inversion = False
+            else:
+                cbook._warn_external(
+                    "Data has no positive values, and therefore cannot be "
+                    "log-scaled.")
+                return 1, 10
+
         if vmin <= 0:
             vmin = minpos
+            # Don't preserve inversion since vmin was replaced
+            preserve_inversion = False
+
         if vmin == vmax:
             vmin = _decade_less(vmin, self._base)
             vmax = _decade_greater(vmax, self._base)
+
+        # If the original input was inverted (vmin > vmax) and both 
+        # endpoints are still positive, don't swap to preserve inversion.
+        # Otherwise, swap to make vmin < vmax (standard behavior).
+        if not preserve_inversion and vmin > vmax:
+            vmin, vmax = vmax, vmin
+
         return vmin, vmax
 
 

</candidate_a_patch>
<candidate_b_patch>
diff --git a/lib/matplotlib/ticker.py b/lib/matplotlib/ticker.py
index 209a3f92c..04e2e72ea 100644
--- a/lib/matplotlib/ticker.py
+++ b/lib/matplotlib/ticker.py
@@ -2379,6 +2379,11 @@ class LogLocator(Locator):
         if not np.isfinite(vmin) or not np.isfinite(vmax):
             return 1, 10  # initial range, no data plotted yet
 
+        # Remember if the range was inverted (vmin > vmax), as this might
+        # be intentional for axis inversion. Only consider it intentional
+        # if both endpoints are positive (valid for log scale).
+        inverted = vmin > vmax and vmin > 0 and vmax > 0
+
         if vmin > vmax:
             vmin, vmax = vmax, vmin
         if vmax <= 0:
@@ -2395,6 +2400,11 @@ class LogLocator(Locator):
         if vmin == vmax:
             vmin = _decade_less(vmin, self._base)
             vmax = _decade_greater(vmax, self._base)
+
+        # Restore the original ordering if the user intentionally inverted.
+        if inverted:
+            vmin, vmax = vmax, vmin
+
         return vmin, vmax
 
 

</candidate_b_patch>
<generated_test_proposal>
{
  "test_name": "test_view_limits_reversed_with_nonpositive_endpoint",
  "specification_gap": "Reversed log limits preserve inversion only when both endpoints are positive. If one endpoint is non-positive and must be replaced, repairing it must not accidentally create an inverted interval.",
  "input_description": "Plot x data with minimum-positive value 10, use a logarithmic x-axis, then ask its LogLocator for view limits from the reversed boundary pair (1, 0).",
  "expected_output": "The invalid zero endpoint is replaced by the axis minimum-positive value, producing the ordered limits (1, 10).",
  "expected_passing_candidate": "candidate_a",
  "why_interesting": "candidate_b swaps the endpoints before replacing zero, yielding (10, 1) and manufacturing an inversion from an invalid endpoint. candidate_a repairs first and then restores standard ordering.",
  "test_patch": "diff --git a/lib/matplotlib/tests/test_ticker.py b/lib/matplotlib/tests/test_ticker.py\nindex ed61f9b11..ef2f3eed7 100644\n--- a/lib/matplotlib/tests/test_ticker.py\n+++ b/lib/matplotlib/tests/test_ticker.py\n@@ -202,6 +202,14 @@ class TestLogLocator(object):\n         loc = mticker.LogLocator(base=2)\n         test_value = np.array([0.5, 1., 2., 4., 8., 16., 32., 64., 128., 256.])\n         assert_almost_equal(loc.tick_values(1, 100), test_value)\n+\n+    def test_view_limits_reversed_with_nonpositive_endpoint(self):\n+        fig, ax = plt.subplots()\n+        ax.plot([10, 100], [1, 2])\n+        ax.set_xscale('log')\n+\n+        locator = ax.xaxis.get_major_locator()\n+        assert locator.view_limits(1, 0) == (1, 10)\n \n     def test_switch_to_autolocator(self):\n         loc = mticker.LogLocator(subs=\"all\")\n",
  "test_command": "cd /testbed && python -m pytest -q lib/matplotlib/tests/test_ticker.py::TestLogLocator::test_view_limits_reversed_with_nonpositive_endpoint"
}
</generated_test_proposal>
<candidate_execution_summary>
{
  "attempt": 1,
  "separates": true,
  "required_passing_candidate": "candidate_a",
  "meets_acceptance_rule": true,
  "results": [
    {
      "label": "candidate_a",
      "returncode": 0,
      "passed": true,
      "test_ran": true,
      "duration_seconds": 3.338,
      "log_path": "02_execution/attempt_01/candidate_a.log"
    },
    {
      "label": "candidate_b",
      "returncode": 1,
      "passed": false,
      "test_ran": true,
      "duration_seconds": 3.396,
      "log_path": "02_execution/attempt_01/candidate_b.log"
    }
  ]
}
</candidate_execution_summary>
<candidate_a_execution_log>
[pipeline] checking candidate patch
/inputs/candidate.patch:45: trailing whitespace.
        # If the original input was inverted (vmin > vmax) and both 
warning: 1 line adds whitespace errors.
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
lib/matplotlib/tests/test_ticker.py::TestLogLocator::test_view_limits_reversed_with_nonpositive_endpoint
  /opt/miniconda3/envs/testbed/lib/python3.8/site-packages/setuptools/_distutils/version.py:337: DeprecationWarning: distutils Version classes are deprecated. Use packaging.version instead.
    other = LooseVersion(other)

lib/matplotlib/tests/test_ticker.py::TestLogLocator::test_view_limits_reversed_with_nonpositive_endpoint
  /testbed/lib/matplotlib/__init__.py:332: DeprecationWarning: distutils Version classes are deprecated. Use packaging.version instead.
    version = LooseVersion(match.group(1))

-- Docs: https://docs.pytest.org/en/stable/how-to/capture-warnings.html
1 passed, 12 warnings in 1.95s
[pipeline] test_exit_code=0

</candidate_a_execution_log>
<candidate_b_execution_log>
[pipeline] checking candidate patch
[pipeline] checking generated test patch
[pipeline] executing generated test command
Matplotlib is not built with the correct FreeType version to run tests.  Set local_freetype=True in setup.cfg and rebuild. Expect many image comparison failures below. Expected freetype version 2.6.1. Found freetype version 2.11.1. Freetype build type is not local
F                                                                        [100%]
=================================== FAILURES ===================================
______ TestLogLocator.test_view_limits_reversed_with_nonpositive_endpoint ______

self = <matplotlib.tests.test_ticker.TestLogLocator object at 0x7c92baf2f3a0>

    def test_view_limits_reversed_with_nonpositive_endpoint(self):
        fig, ax = plt.subplots()
        ax.plot([10, 100], [1, 2])
        ax.set_xscale('log')
    
        locator = ax.xaxis.get_major_locator()
>       assert locator.view_limits(1, 0) == (1, 10)
E       assert (10.0, 1.0) == (1, 10)
E         
E         At index 0 diff: 10.0 != 1
E         Use -v to get more diff

lib/matplotlib/tests/test_ticker.py:212: AssertionError
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
lib/matplotlib/tests/test_ticker.py::TestLogLocator::test_view_limits_reversed_with_nonpositive_endpoint
  /opt/miniconda3/envs/testbed/lib/python3.8/site-packages/setuptools/_distutils/version.py:337: DeprecationWarning: distutils Version classes are deprecated. Use packaging.version instead.
    other = LooseVersion(other)

lib/matplotlib/tests/test_ticker.py::TestLogLocator::test_view_limits_reversed_with_nonpositive_endpoint
  /testbed/lib/matplotlib/__init__.py:332: DeprecationWarning: distutils Version classes are deprecated. Use packaging.version instead.
    version = LooseVersion(match.group(1))

-- Docs: https://docs.pytest.org/en/stable/how-to/capture-warnings.html
=========================== short test summary info ============================
FAILED lib/matplotlib/tests/test_ticker.py::TestLogLocator::test_view_limits_reversed_with_nonpositive_endpoint
1 failed, 12 warnings in 2.06s
[pipeline] test_exit_code=1

</candidate_b_execution_log>
<official_gold_patch>
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

</official_gold_patch>
<gold_execution_summary>
{
  "label": "gold",
  "returncode": 1,
  "passed": false,
  "test_ran": true,
  "duration_seconds": 3.561,
  "log_path": "gold_execution/gold.log"
}
</gold_execution_summary>
<gold_failure_contract>
{
  "kind": "behavioral_test_failure",
  "required_substrings": [
    "test_view_limits_reversed_with_nonpositive_endpoint"
  ],
  "required_any_substrings": [
    "AssertionError",
    "FAILED",
    " failed,"
  ],
  "forbidden_substrings": [
    "collected 0 items",
    "no tests ran",
    "ERROR collecting",
    "command not found",
    "No such file or directory",
    "Could not find a version that satisfies the requirement",
    "Temporary failure in name resolution"
  ],
  "reference_returncode": 1,
  "reference_log_sha256": "7bc029ea67d920b3909261c55aecd98dd7f97cff56bd3ec82d5bee7e3efa05ee"
}
</gold_failure_contract>
<gold_execution_log>
[pipeline] checking candidate patch
[pipeline] checking generated test patch
[pipeline] executing generated test command
Matplotlib is not built with the correct FreeType version to run tests.  Set local_freetype=True in setup.cfg and rebuild. Expect many image comparison failures below. Expected freetype version 2.6.1. Found freetype version 2.11.1. Freetype build type is not local
F                                                                        [100%]
=================================== FAILURES ===================================
______ TestLogLocator.test_view_limits_reversed_with_nonpositive_endpoint ______

self = <matplotlib.tests.test_ticker.TestLogLocator object at 0x7141ed288280>

    def test_view_limits_reversed_with_nonpositive_endpoint(self):
        fig, ax = plt.subplots()
        ax.plot([10, 100], [1, 2])
        ax.set_xscale('log')
    
        locator = ax.xaxis.get_major_locator()
>       assert locator.view_limits(1, 0) == (1, 10)
E       assert (10.0, 1.0) == (1, 10)
E         
E         At index 0 diff: 10.0 != 1
E         Use -v to get more diff

lib/matplotlib/tests/test_ticker.py:212: AssertionError
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
lib/matplotlib/tests/test_ticker.py::TestLogLocator::test_view_limits_reversed_with_nonpositive_endpoint
  /opt/miniconda3/envs/testbed/lib/python3.8/site-packages/setuptools/_distutils/version.py:337: DeprecationWarning: distutils Version classes are deprecated. Use packaging.version instead.
    other = LooseVersion(other)

lib/matplotlib/tests/test_ticker.py::TestLogLocator::test_view_limits_reversed_with_nonpositive_endpoint
  /testbed/lib/matplotlib/__init__.py:332: DeprecationWarning: distutils Version classes are deprecated. Use packaging.version instead.
    version = LooseVersion(match.group(1))

-- Docs: https://docs.pytest.org/en/stable/how-to/capture-warnings.html
=========================== short test summary info ============================
FAILED lib/matplotlib/tests/test_ticker.py::TestLogLocator::test_view_limits_reversed_with_nonpositive_endpoint
1 failed, 12 warnings in 2.05s
[pipeline] test_exit_code=1

</gold_execution_log>
