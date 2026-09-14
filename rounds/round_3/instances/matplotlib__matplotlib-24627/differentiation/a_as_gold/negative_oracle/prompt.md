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
cla(), clf() should unset the `.axes` and `.figure` attributes of deparented artists
mpl2.0b3: Removing an artist from its axes unsets its `.axes` attribute, but clearing the axes does not do so.

```
In [11]: f, a = plt.subplots(); l, = a.plot([1, 2]); l.remove(); print(l.axes)
None

In [12]: f, a = plt.subplots(); l, = a.plot([1, 2]); a.cla(); print(l.axes)
Axes(0.125,0.11;0.775x0.77)
```


</issue_statement>
<candidate_a_patch>
diff --git a/lib/matplotlib/axes/_base.py b/lib/matplotlib/axes/_base.py
index 90b0bf117e..46a99c6a7c 100644
--- a/lib/matplotlib/axes/_base.py
+++ b/lib/matplotlib/axes/_base.py
@@ -1307,6 +1307,15 @@ class _AxesBase(martist.Artist):
         self._get_lines = _process_plot_var_args(self)
         self._get_patches_for_fill = _process_plot_var_args(self, 'fill')
 
+        # Deparent all children and containers before clearing.
+        for child in self._children:
+            child.axes = None
+            child.figure = None
+        if hasattr(self, "containers"):
+            for container in self.containers:
+                container.axes = None
+                container.figure = None
+
         self._gridOn = mpl.rcParams['axes.grid']
         self._children = []
         self._mouseover_set = _OrderedSet()

</candidate_a_patch>
<candidate_b_patch>
diff --git a/lib/matplotlib/axes/_base.py b/lib/matplotlib/axes/_base.py
index 90b0bf117e..b51263b6d4 100644
--- a/lib/matplotlib/axes/_base.py
+++ b/lib/matplotlib/axes/_base.py
@@ -1308,6 +1308,9 @@ class _AxesBase(martist.Artist):
         self._get_patches_for_fill = _process_plot_var_args(self, 'fill')
 
         self._gridOn = mpl.rcParams['axes.grid']
+        for child in self._children:
+            child.axes = None
+            child.figure = None
         self._children = []
         self._mouseover_set = _OrderedSet()
         self.child_axes = []

</candidate_b_patch>
<generated_test_proposal>
{
  "test_name": "test_clf_deparents_container",
  "specification_gap": "Figure clearing must deparent Axes containers as well as ordinary child artists. A public aggregate plotting result must have its `.axes` and `.figure` references unset after `clf()`.",
  "input_description": "Create a Figure and Axes, obtain a BarContainer from `Axes.bar([0], [1])`, and call `Figure.clf()`.",
  "expected_output": "The returned BarContainer exposes `axes is None` and `figure is None`; the targeted pytest command exits 0.",
  "expected_passing_candidate": "candidate_a",
  "why_interesting": "BarContainer is tracked separately from the Axes' ordinary children. This exercises both an alternate public result type and the issue's `clf()` entry point, exposing the generated candidate's child-only cleanup.",
  "test_patch": "diff --git a/lib/matplotlib/tests/test_figure.py b/lib/matplotlib/tests/test_figure.py\n--- a/lib/matplotlib/tests/test_figure.py\n+++ b/lib/matplotlib/tests/test_figure.py\n@@ -767,9 +767,19 @@ def test_figure_clear(clear_meth):\n     getattr(fig, clear_meth)()\n     assert fig.subfigs == []\n     assert fig.axes == []\n \n \n+def test_clf_deparents_container():\n+    fig, ax = plt.subplots()\n+    container = ax.bar([0], [1])\n+\n+    fig.clf()\n+\n+    assert container.axes is None\n+    assert container.figure is None\n+\n+\n def test_clf_not_redefined():\n     for klass in FigureBase.__subclasses__():\n         # check that subclasses do not get redefined in our Figure subclasses\n         assert 'clf' not in klass.__dict__\n",
  "test_command": "cd /testbed && python -m pytest -q lib/matplotlib/tests/test_figure.py::test_clf_deparents_container"
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
      "duration_seconds": 3.15,
      "log_path": "02_execution/attempt_01/candidate_a.log"
    },
    {
      "label": "candidate_b",
      "returncode": 1,
      "passed": false,
      "test_ran": true,
      "duration_seconds": 2.883,
      "log_path": "02_execution/attempt_01/candidate_b.log"
    }
  ]
}
</candidate_execution_summary>
<candidate_a_execution_log>
[pipeline] checking candidate patch
[pipeline] checking generated test patch
[pipeline] executing generated test command
.                                                                        [100%]
1 passed in 1.93s
[pipeline] test_exit_code=0

</candidate_a_execution_log>
<candidate_b_execution_log>
[pipeline] checking candidate patch
[pipeline] checking generated test patch
[pipeline] executing generated test command
F                                                                        [100%]
=================================== FAILURES ===================================
_________________________ test_clf_deparents_container _________________________

    def test_clf_deparents_container():
        fig, ax = plt.subplots()
        container = ax.bar([0], [1])
    
        fig.clf()
    
>       assert container.axes is None
E       AttributeError: 'BarContainer' object has no attribute 'axes'

lib/matplotlib/tests/test_figure.py:778: AttributeError
=========================== short test summary info ============================
FAILED lib/matplotlib/tests/test_figure.py::test_clf_deparents_container - At...
1 failed in 1.84s
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
  "duration_seconds": 2.883,
  "log_path": "gold_execution/gold.log"
}
</gold_execution_summary>
<gold_failure_contract>
{
  "kind": "behavioral_test_failure",
  "required_substrings": [
    "test_clf_deparents_container"
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
  "reference_log_sha256": "151e16f686bb632ab0fc5d993f586a8079f170adb4d1b2bb2fd8fe5e18cfec2f"
}
</gold_failure_contract>
<gold_execution_log>
[pipeline] checking candidate patch
[pipeline] checking generated test patch
[pipeline] executing generated test command
F                                                                        [100%]
=================================== FAILURES ===================================
_________________________ test_clf_deparents_container _________________________

    def test_clf_deparents_container():
        fig, ax = plt.subplots()
        container = ax.bar([0], [1])
    
        fig.clf()
    
>       assert container.axes is None
E       AttributeError: 'BarContainer' object has no attribute 'axes'

lib/matplotlib/tests/test_figure.py:778: AttributeError
=========================== short test summary info ============================
FAILED lib/matplotlib/tests/test_figure.py::test_clf_deparents_container - At...
1 failed in 1.82s
[pipeline] test_exit_code=1

</gold_execution_log>
