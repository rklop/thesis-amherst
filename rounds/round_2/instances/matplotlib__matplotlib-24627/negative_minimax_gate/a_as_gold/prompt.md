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
diff --git a/lib/matplotlib/axes/_base.py b/lib/matplotlib/axes/_base.py
index 90b0bf117e..14c9500cc1 100644
--- a/lib/matplotlib/axes/_base.py
+++ b/lib/matplotlib/axes/_base.py
@@ -1308,12 +1308,23 @@ class _AxesBase(martist.Artist):
         self._get_patches_for_fill = _process_plot_var_args(self, 'fill')
 
         self._gridOn = mpl.rcParams['axes.grid']
+        for artist in list(self._children):
+            artist.remove()
         self._children = []
         self._mouseover_set = _OrderedSet()
         self.child_axes = []
         self._current_image = None  # strictly for pyplot via _sci, _gci
         self._projection_init = None  # strictly for pyplot.subplot
         self.legend_ = None
+        if hasattr(self, "containers"):
+            # Also clear containers' children axes/figure refs, to match
+            # behavior of children cleared above.
+            for container in self.containers:
+                for c in container:
+                    if hasattr(c, 'axes'):
+                        c.axes = None
+                    if hasattr(c, 'figure'):
+                        c.figure = None
         self.containers = []
 
         self.grid(False)  # Disable grid on init to use rcParameter

</candidate_a_patch>
<candidate_b_patch>
diff --git a/lib/matplotlib/axes/_base.py b/lib/matplotlib/axes/_base.py
index 90b0bf117e..3c19531795 100644
--- a/lib/matplotlib/axes/_base.py
+++ b/lib/matplotlib/axes/_base.py
@@ -1308,12 +1308,20 @@ class _AxesBase(martist.Artist):
         self._get_patches_for_fill = _process_plot_var_args(self, 'fill')
 
         self._gridOn = mpl.rcParams['axes.grid']
+        for child in self._children:
+            child.axes = None
+            child.figure = None
         self._children = []
         self._mouseover_set = _OrderedSet()
         self.child_axes = []
         self._current_image = None  # strictly for pyplot via _sci, _gci
         self._projection_init = None  # strictly for pyplot.subplot
         self.legend_ = None
+        if hasattr(self, 'containers'):
+            for container in self.containers:
+                for child in container.get_children():
+                    child.axes = None
+                    child.figure = None
         self.containers = []
 
         self.grid(False)  # Disable grid on init to use rcParameter

</candidate_b_patch>
<generated_test_proposal>
{
  "test_name": "test_clear_fully_removes_artist",
  "specification_gap": "Axes.clear() must complete the normal Artist.remove() lifecycle, not merely set the artist's parent attributes to None. A subsequent remove() is therefore a repeated removal.",
  "input_description": "Create a standard Line2D with Axes.plot(), clear the axes, inspect its public axes and figure attributes, then call its public remove() method again.",
  "expected_output": "After clear(), line.axes and line.figure are None, and the subsequent remove() raises ValueError because the artist has already been removed.",
  "expected_passing_candidate": "candidate_a",
  "why_interesting": "This distinguishes complete removal from superficial parent-field assignment using only a standard artist and public operations. candidate_a removes each child normally; candidate_b leaves the old removal association usable, so the subsequent remove() incorrectly succeeds.",
  "test_patch": "diff --git a/lib/matplotlib/tests/test_axes.py b/lib/matplotlib/tests/test_axes.py\n--- a/lib/matplotlib/tests/test_axes.py\n+++ b/lib/matplotlib/tests/test_axes.py\n@@ -486,5 +486,16 @@ def test_inverted_cla():\n     # clean up\n     plt.close(fig)\n \n \n+def test_clear_fully_removes_artist():\n+    fig, ax = plt.subplots()\n+    line, = ax.plot([0, 1])\n+    ax.clear()\n+\n+    assert line.axes is None\n+    assert line.figure is None\n+    with pytest.raises(ValueError):\n+        line.remove()\n+\n+\n def test_subclass_clear_cla():\n",
  "test_command": "python -m pytest -q lib/matplotlib/tests/test_axes.py::test_clear_fully_removes_artist"
}
</generated_test_proposal>
<candidate_execution_summary>
{
  "attempt": 2,
  "separates": true,
  "required_passing_candidate": "candidate_a",
  "meets_acceptance_rule": true,
  "results": [
    {
      "label": "candidate_a",
      "returncode": 0,
      "passed": true,
      "test_ran": true,
      "duration_seconds": 3.226,
      "log_path": "02_execution/attempt_02/candidate_a.log"
    },
    {
      "label": "candidate_b",
      "returncode": 1,
      "passed": false,
      "test_ran": true,
      "duration_seconds": 3.341,
      "log_path": "02_execution/attempt_02/candidate_b.log"
    }
  ]
}
</candidate_execution_summary>
<candidate_a_execution_log>
[pipeline] checking candidate patch
[pipeline] checking generated test patch
[pipeline] executing generated test command
.                                                                        [100%]
1 passed in 2.17s
[pipeline] test_exit_code=0

</candidate_a_execution_log>
<candidate_b_execution_log>
[pipeline] checking candidate patch
[pipeline] checking generated test patch
[pipeline] executing generated test command
F                                                                        [100%]
=================================== FAILURES ===================================
_______________________ test_clear_fully_removes_artist ________________________

    def test_clear_fully_removes_artist():
        fig, ax = plt.subplots()
        line, = ax.plot([0, 1])
        ax.clear()
    
        assert line.axes is None
        assert line.figure is None
>       with pytest.raises(ValueError):
E       Failed: DID NOT RAISE <class 'ValueError'>

lib/matplotlib/tests/test_axes.py:497: Failed
=========================== short test summary info ============================
FAILED lib/matplotlib/tests/test_axes.py::test_clear_fully_removes_artist - F...
1 failed in 2.30s
[pipeline] test_exit_code=1

</candidate_b_execution_log>
<official_gold_patch>
diff --git a/lib/matplotlib/axes/_base.py b/lib/matplotlib/axes/_base.py
--- a/lib/matplotlib/axes/_base.py
+++ b/lib/matplotlib/axes/_base.py
@@ -1315,7 +1315,9 @@ def __clear(self):
         self._get_patches_for_fill = _process_plot_var_args(self, 'fill')
 
         self._gridOn = mpl.rcParams['axes.grid']
-        self._children = []
+        old_children, self._children = self._children, []
+        for chld in old_children:
+            chld.axes = chld.figure = None
         self._mouseover_set = _OrderedSet()
         self.child_axes = []
         self._current_image = None  # strictly for pyplot via _sci, _gci

</official_gold_patch>
<gold_execution_summary>
{
  "label": "gold",
  "returncode": 1,
  "passed": false,
  "test_ran": true,
  "duration_seconds": 3.356,
  "log_path": "gold_execution/gold.log"
}
</gold_execution_summary>
<gold_failure_contract>
{
  "kind": "behavioral_test_failure",
  "required_substrings": [
    "test_clear_fully_removes_artist"
  ],
  "required_any_substrings": [
    "FAILED"
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
  "reference_log_sha256": "d8ac3b1fb845c646a2ab85d8d7c4ed9fd31ecfe6877663a07758ec1863bd5890"
}
</gold_failure_contract>
<gold_execution_log>
[pipeline] checking candidate patch
[pipeline] checking generated test patch
[pipeline] executing generated test command
F                                                                        [100%]
=================================== FAILURES ===================================
_______________________ test_clear_fully_removes_artist ________________________

    def test_clear_fully_removes_artist():
        fig, ax = plt.subplots()
        line, = ax.plot([0, 1])
        ax.clear()
    
        assert line.axes is None
        assert line.figure is None
>       with pytest.raises(ValueError):
E       Failed: DID NOT RAISE <class 'ValueError'>

lib/matplotlib/tests/test_axes.py:497: Failed
=========================== short test summary info ============================
FAILED lib/matplotlib/tests/test_axes.py::test_clear_fully_removes_artist - F...
1 failed in 2.30s
[pipeline] test_exit_code=1

</gold_execution_log>
