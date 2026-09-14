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
ResolverMatch.__repr__() doesn't handle functools.partial() nicely.
Description
	 
		(last modified by Nick Pope)
	 
When a partial function is passed as the view, the __repr__ shows the func argument as functools.partial which isn't very helpful, especially as it doesn't reveal the underlying function or arguments provided.
Because a partial function also has arguments provided up front, we need to handle those specially so that they are accessible in __repr__.
ISTM that we can simply unwrap functools.partial objects in ResolverMatch.__init__().

</issue_statement>
<candidate_a_patch>
diff --git a/django/urls/resolvers.py b/django/urls/resolvers.py
index 9b00e24509..77e754353a 100644
--- a/django/urls/resolvers.py
+++ b/django/urls/resolvers.py
@@ -45,12 +45,22 @@ class ResolverMatch:
         self.namespaces = [x for x in namespaces if x] if namespaces else []
         self.namespace = ':'.join(self.namespaces)
 
-        if not hasattr(func, '__name__'):
+        # Unwrap functools.partial objects.
+        self._func = func
+        self._partial_args = ()
+        self._partial_kwargs = {}
+        while isinstance(func, functools.partial):
+            self._partial_args = func.args + self._partial_args
+            self._partial_kwargs = {**func.keywords, **self._partial_kwargs}
+            func = func.func
+        self._unwrapped_func = func
+
+        if not hasattr(self._unwrapped_func, '__name__'):
             # A class-based view
-            self._func_path = func.__class__.__module__ + '.' + func.__class__.__name__
+            self._func_path = self._unwrapped_func.__class__.__module__ + '.' + self._unwrapped_func.__class__.__name__
         else:
             # A function-based view
-            self._func_path = func.__module__ + '.' + func.__name__
+            self._func_path = self._unwrapped_func.__module__ + '.' + self._unwrapped_func.__name__
 
         view_path = url_name or self._func_path
         self.view_name = ':'.join(self.namespaces + [view_path])
@@ -59,8 +69,17 @@ class ResolverMatch:
         return (self.func, self.args, self.kwargs)[index]
 
     def __repr__(self):
-        return "ResolverMatch(func=%s, args=%s, kwargs=%s, url_name=%s, app_names=%s, namespaces=%s, route=%s)" % (
-            self._func_path, self.args, self.kwargs, self.url_name,
+        func = self._func
+        if isinstance(func, functools.partial):
+            func = functools.partial(
+                self._unwrapped_func,
+                *self._partial_args,
+                **self._partial_kwargs,
+            )
+        else:
+            func = self._func_path
+        return "ResolverMatch(func=%s, args=%s, kwargs=%s, url_name=%r, app_names=%r, namespaces=%r, route=%r)" % (
+            func, self.args, self.kwargs, self.url_name,
             self.app_names, self.namespaces, self.route,
         )
 

</candidate_a_patch>
<candidate_b_patch>
diff --git a/django/urls/resolvers.py b/django/urls/resolvers.py
index 9b00e24509..3b0510e201 100644
--- a/django/urls/resolvers.py
+++ b/django/urls/resolvers.py
@@ -45,6 +45,10 @@ class ResolverMatch:
         self.namespaces = [x for x in namespaces if x] if namespaces else []
         self.namespace = ':'.join(self.namespaces)
 
+        # Unwrap functools.partial objects.
+        while isinstance(func, functools.partial):
+            func = func.func
+
         if not hasattr(func, '__name__'):
             # A class-based view
             self._func_path = func.__class__.__module__ + '.' + func.__class__.__name__
@@ -59,8 +63,13 @@ class ResolverMatch:
         return (self.func, self.args, self.kwargs)[index]
 
     def __repr__(self):
-        return "ResolverMatch(func=%s, args=%s, kwargs=%s, url_name=%s, app_names=%s, namespaces=%s, route=%s)" % (
-            self._func_path, self.args, self.kwargs, self.url_name,
+        # Handle functools.partial objects.
+        if isinstance(self.func, functools.partial):
+            func_repr = repr(self.func)
+        else:
+            func_repr = self._func_path
+        return "ResolverMatch(func=%s, args=%s, kwargs=%s, url_name=%r, app_names=%s, namespaces=%s, route=%r)" % (
+            func_repr, self.args, self.kwargs, self.url_name,
             self.app_names, self.namespaces, self.route,
         )
 

</candidate_b_patch>
<generated_test_proposal>
{
  "test_name": "test_repr_partial_subclass",
  "specification_gap": "ResolverMatch must expose the underlying callable and pre-bound arguments for any functools.partial instance, including subclasses whose own repr is opaque.",
  "input_description": "Create an opaque-repr functools.partial subclass wrapping empty_view with positional argument 'preset' and keyword argument template_name='template.html', then pass it to ResolverMatch.",
  "expected_output": "repr(ResolverMatch(...)) contains 'empty_view', 'preset', and \"template_name='template.html'\" despite the partial subclass hiding those details in its own repr.",
  "expected_passing_candidate": "candidate_a",
  "why_interesting": "This alternate accepted callable type directly tests the issue\u2019s diagnostic-information contract. Candidate A reconstructs an informative partial representation from its semantic fields, while candidate B delegates to the opaque object repr and loses the target and bound arguments.",
  "test_patch": "diff --git a/tests/urlpatterns_reverse/tests.py b/tests/urlpatterns_reverse/tests.py\n--- a/tests/urlpatterns_reverse/tests.py\n+++ b/tests/urlpatterns_reverse/tests.py\n@@ -1,6 +1,7 @@\n \"\"\"\n Unit tests for reverse URL lookups.\n \"\"\"\n+import functools\n import sys\n import threading\n \n@@ -1140,10 +1141,23 @@ class ResolverMatchTests(SimpleTestCase):\n     def test_repr(self):\n         self.assertEqual(\n             repr(resolve('/no_kwargs/42/37/')),\n             \"ResolverMatch(func=urlpatterns_reverse.views.empty_view, \"\n             \"args=('42', '37'), kwargs={}, url_name=no-kwargs, app_names=[], \"\n             \"namespaces=[], route=^no_kwargs/([0-9]+)/([0-9]+)/$)\",\n         )\n \n+    def test_repr_partial_subclass(self):\n+        class OpaquePartial(functools.partial):\n+            def __repr__(self):\n+                return '<opaque partial>'\n+\n+        func = OpaquePartial(\n+            empty_view, 'preset', template_name='template.html',\n+        )\n+        representation = repr(ResolverMatch(func, (), {}))\n+        for value in ('empty_view', \"'preset'\", \"template_name='template.html'\"):\n+            with self.subTest(value=value):\n+                self.assertIn(value, representation)\n+\n \n @override_settings(ROOT_URLCONF='urlpatterns_reverse.erroneous_urls')\n class ErroneousViewTests(SimpleTestCase):\n",
  "test_command": "cd /testbed && ./tests/runtests.py urlpatterns_reverse.tests.ResolverMatchTests.test_repr_partial_subclass"
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
      "duration_seconds": 1.472,
      "log_path": "02_execution/attempt_01/candidate_a.log"
    },
    {
      "label": "candidate_b",
      "returncode": 1,
      "passed": false,
      "test_ran": true,
      "duration_seconds": 1.443,
      "log_path": "02_execution/attempt_01/candidate_b.log"
    }
  ]
}
</candidate_execution_summary>
<candidate_a_execution_log>
[pipeline] checking candidate patch
[pipeline] checking generated test patch
[pipeline] executing generated test command
.Testing against Django installed in '/testbed/django' with up to 24 processes
System check identified no issues (0 silenced).

----------------------------------------------------------------------
Ran 1 test in 0.000s

OK
[pipeline] test_exit_code=0

</candidate_a_execution_log>
<candidate_b_execution_log>
[pipeline] checking candidate patch
[pipeline] checking generated test patch
[pipeline] executing generated test command
Testing against Django installed in '/testbed/django' with up to 24 processes
System check identified no issues (0 silenced).

======================================================================
FAIL: test_repr_partial_subclass (urlpatterns_reverse.tests.ResolverMatchTests) (value='empty_view')
----------------------------------------------------------------------
Traceback (most recent call last):
  File "/testbed/tests/urlpatterns_reverse/tests.py", line 1160, in test_repr_partial_subclass
    self.assertIn(value, representation)
AssertionError: 'empty_view' not found in 'ResolverMatch(func=<opaque partial>, args=(), kwargs={}, url_name=None, app_names=[], namespaces=[], route=None)'

======================================================================
FAIL: test_repr_partial_subclass (urlpatterns_reverse.tests.ResolverMatchTests) (value="'preset'")
----------------------------------------------------------------------
Traceback (most recent call last):
  File "/testbed/tests/urlpatterns_reverse/tests.py", line 1160, in test_repr_partial_subclass
    self.assertIn(value, representation)
AssertionError: "'preset'" not found in 'ResolverMatch(func=<opaque partial>, args=(), kwargs={}, url_name=None, app_names=[], namespaces=[], route=None)'

======================================================================
FAIL: test_repr_partial_subclass (urlpatterns_reverse.tests.ResolverMatchTests) (value="template_name='template.html'")
----------------------------------------------------------------------
Traceback (most recent call last):
  File "/testbed/tests/urlpatterns_reverse/tests.py", line 1160, in test_repr_partial_subclass
    self.assertIn(value, representation)
AssertionError: "template_name='template.html'" not found in 'ResolverMatch(func=<opaque partial>, args=(), kwargs={}, url_name=None, app_names=[], namespaces=[], route=None)'

----------------------------------------------------------------------
Ran 1 test in 0.000s

FAILED (failures=3)
[pipeline] test_exit_code=1

</candidate_b_execution_log>
<official_gold_patch>
diff --git a/django/urls/resolvers.py b/django/urls/resolvers.py
--- a/django/urls/resolvers.py
+++ b/django/urls/resolvers.py
@@ -59,9 +59,16 @@ def __getitem__(self, index):
         return (self.func, self.args, self.kwargs)[index]
 
     def __repr__(self):
-        return "ResolverMatch(func=%s, args=%s, kwargs=%s, url_name=%s, app_names=%s, namespaces=%s, route=%s)" % (
-            self._func_path, self.args, self.kwargs, self.url_name,
-            self.app_names, self.namespaces, self.route,
+        if isinstance(self.func, functools.partial):
+            func = repr(self.func)
+        else:
+            func = self._func_path
+        return (
+            'ResolverMatch(func=%s, args=%r, kwargs=%r, url_name=%r, '
+            'app_names=%r, namespaces=%r, route=%r)' % (
+                func, self.args, self.kwargs, self.url_name,
+                self.app_names, self.namespaces, self.route,
+            )
         )
 
 

</official_gold_patch>
<gold_execution_summary>
{
  "label": "gold",
  "returncode": 1,
  "passed": false,
  "test_ran": true,
  "duration_seconds": 1.545,
  "log_path": "gold_execution/gold.log"
}
</gold_execution_summary>
<gold_failure_contract>
{
  "kind": "behavioral_test_failure",
  "required_substrings": [
    "test_repr_partial_subclass"
  ],
  "required_any_substrings": [
    "AssertionError",
    "FAILED",
    "FAIL:"
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
  "reference_log_sha256": "2f29f79af406c2005f7c3845359420594d61c1689c8eb09cd8277916b916f023"
}
</gold_failure_contract>
<gold_execution_log>
[pipeline] checking candidate patch
[pipeline] checking generated test patch
[pipeline] executing generated test command
Testing against Django installed in '/testbed/django' with up to 24 processes
System check identified no issues (0 silenced).

======================================================================
FAIL: test_repr_partial_subclass (urlpatterns_reverse.tests.ResolverMatchTests) (value='empty_view')
----------------------------------------------------------------------
Traceback (most recent call last):
  File "/testbed/tests/urlpatterns_reverse/tests.py", line 1160, in test_repr_partial_subclass
    self.assertIn(value, representation)
AssertionError: 'empty_view' not found in 'ResolverMatch(func=<opaque partial>, args=(), kwargs={}, url_name=None, app_names=[], namespaces=[], route=None)'

======================================================================
FAIL: test_repr_partial_subclass (urlpatterns_reverse.tests.ResolverMatchTests) (value="'preset'")
----------------------------------------------------------------------
Traceback (most recent call last):
  File "/testbed/tests/urlpatterns_reverse/tests.py", line 1160, in test_repr_partial_subclass
    self.assertIn(value, representation)
AssertionError: "'preset'" not found in 'ResolverMatch(func=<opaque partial>, args=(), kwargs={}, url_name=None, app_names=[], namespaces=[], route=None)'

======================================================================
FAIL: test_repr_partial_subclass (urlpatterns_reverse.tests.ResolverMatchTests) (value="template_name='template.html'")
----------------------------------------------------------------------
Traceback (most recent call last):
  File "/testbed/tests/urlpatterns_reverse/tests.py", line 1160, in test_repr_partial_subclass
    self.assertIn(value, representation)
AssertionError: "template_name='template.html'" not found in 'ResolverMatch(func=<opaque partial>, args=(), kwargs={}, url_name=None, app_names=[], namespaces=[], route=None)'

----------------------------------------------------------------------
Ran 1 test in 0.000s

FAILED (failures=3)
[pipeline] test_exit_code=1

</gold_execution_log>
