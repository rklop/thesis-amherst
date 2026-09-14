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
ResolverMatch.__repr__() doesn't handle functools.partial() nicely.
Description
	 
		(last modified by Nick Pope)
	 
When a partial function is passed as the view, the __repr__ shows the func argument as functools.partial which isn't very helpful, especially as it doesn't reveal the underlying function or arguments provided.
Because a partial function also has arguments provided up front, we need to handle those specially so that they are accessible in __repr__.
ISTM that we can simply unwrap functools.partial objects in ResolverMatch.__init__().

</issue_statement>

<original_test_patch>
diff --git a/tests/urlpatterns_reverse/tests.py b/tests/urlpatterns_reverse/tests.py
--- a/tests/urlpatterns_reverse/tests.py
+++ b/tests/urlpatterns_reverse/tests.py
@@ -1141,10 +1141,30 @@ def test_repr(self):
         self.assertEqual(
             repr(resolve('/no_kwargs/42/37/')),
             "ResolverMatch(func=urlpatterns_reverse.views.empty_view, "
-            "args=('42', '37'), kwargs={}, url_name=no-kwargs, app_names=[], "
-            "namespaces=[], route=^no_kwargs/([0-9]+)/([0-9]+)/$)",
+            "args=('42', '37'), kwargs={}, url_name='no-kwargs', app_names=[], "
+            "namespaces=[], route='^no_kwargs/([0-9]+)/([0-9]+)/$')",
         )
 
+    @override_settings(ROOT_URLCONF='urlpatterns_reverse.urls')
+    def test_repr_functools_partial(self):
+        tests = [
+            ('partial', 'template.html'),
+            ('partial_nested', 'nested_partial.html'),
+            ('partial_wrapped', 'template.html'),
+        ]
+        for name, template_name in tests:
+            with self.subTest(name=name):
+                func = (
+                    f"functools.partial({views.empty_view!r}, "
+                    f"template_name='{template_name}')"
+                )
+                self.assertEqual(
+                    repr(resolve(f'/{name}/')),
+                    f"ResolverMatch(func={func}, args=(), kwargs={{}}, "
+                    f"url_name='{name}', app_names=[], namespaces=[], "
+                    f"route='{name}/')",
+                )
+
 
 @override_settings(ROOT_URLCONF='urlpatterns_reverse.erroneous_urls')
 class ErroneousViewTests(SimpleTestCase):

</original_test_patch>

<candidate_a_patch>
diff --git a/django/urls/resolvers.py b/django/urls/resolvers.py
index 9b00e24509..b1a43b9b74 100644
--- a/django/urls/resolvers.py
+++ b/django/urls/resolvers.py
@@ -45,23 +45,38 @@ class ResolverMatch:
         self.namespaces = [x for x in namespaces if x] if namespaces else []
         self.namespace = ':'.join(self.namespaces)
 
+        # Unwrap functools.partial objects.
+        self._func_path = self._get_func_path(func)
+
+        view_path = url_name or self._func_path
+        self.view_name = ':'.join(self.namespaces + [view_path])
+
+    def _get_func_path(self, func):
+        """
+        Return the function path for a view func.
+        """
+        # If it's a partial, unwrap it to get the underlying function.
+        if isinstance(func, functools.partial):
+            func = func.func
         if not hasattr(func, '__name__'):
             # A class-based view
-            self._func_path = func.__class__.__module__ + '.' + func.__class__.__name__
+            return func.__class__.__module__ + '.' + func.__class__.__name__
         else:
             # A function-based view
-            self._func_path = func.__module__ + '.' + func.__name__
-
-        view_path = url_name or self._func_path
-        self.view_name = ':'.join(self.namespaces + [view_path])
+            return func.__module__ + '.' + func.__name__
 
     def __getitem__(self, index):
         return (self.func, self.args, self.kwargs)[index]
 
     def __repr__(self):
         return "ResolverMatch(func=%s, args=%s, kwargs=%s, url_name=%s, app_names=%s, namespaces=%s, route=%s)" % (
-            self._func_path, self.args, self.kwargs, self.url_name,
-            self.app_names, self.namespaces, self.route,
+            repr(self.func) if isinstance(self.func, functools.partial) else self._func_path,
+            self.args,
+            self.kwargs,
+            repr(self.url_name),
+            self.app_names,
+            self.namespaces,
+            repr(self.route),
         )
 
 

</candidate_a_patch>

<candidate_b_patch>
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
 
 

</candidate_b_patch>

<generated_test_proposal>
{
  "test_name": "test_repr_partial_builtin_callable",
  "specification_gap": "A functools.partial may wrap any callable, including a built-in method descriptor without __module__. ResolverMatch must expose the wrapped callable and bound arguments without assuming the wrapped callable is a conventional Python function.",
  "input_description": "Create an unnamed URL pattern whose view is functools.partial(str.join, ',') and resolve the path 'join/'.",
  "expected_output": "Resolution succeeds and repr(match) is \"ResolverMatch(func=functools.partial(<method 'join' of 'str' objects>, ','), args=(), kwargs={}, url_name=None, app_names=[], namespaces=[], route='join/')\". Candidate A instead raises AttributeError while eagerly deriving a function path from str.join.",
  "expected_passing_candidate": "candidate_b",
  "why_interesting": "This covers the public path()/URLPattern.resolve() entry point and a valid callable-type boundary. Candidate A's unwrapping introduces a regression for built-in callables, while candidate B formats the partial directly and preserves resolution.",
  "test_patch": "diff --git a/tests/urlpatterns_reverse/tests.py b/tests/urlpatterns_reverse/tests.py\n--- a/tests/urlpatterns_reverse/tests.py\n+++ b/tests/urlpatterns_reverse/tests.py\n@@ -1,6 +1,7 @@\n \"\"\"\n Unit tests for reverse URL lookups.\n \"\"\"\n+from functools import partial\n import sys\n import threading\n \n@@ -1145,6 +1146,15 @@ class ResolverMatchTests(SimpleTestCase):\n             \"namespaces=[], route=^no_kwargs/([0-9]+)/([0-9]+)/$)\",\n         )\n \n+    def test_repr_partial_builtin_callable(self):\n+        view = partial(str.join, ',')\n+        match = path('join/', view).resolve('join/')\n+        self.assertEqual(\n+            repr(match),\n+            \"ResolverMatch(func=%r, args=(), kwargs={}, url_name=None, \"\n+            \"app_names=[], namespaces=[], route='join/')\" % view,\n+        )\n+\n \n @override_settings(ROOT_URLCONF='urlpatterns_reverse.erroneous_urls')\n class ErroneousViewTests(SimpleTestCase):\n",
  "test_command": "cd /testbed && python tests/runtests.py urlpatterns_reverse.tests.ResolverMatchTests.test_repr_partial_builtin_callable"
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
Testing against Django installed in '/testbed/django' with up to 24 processes
System check identified no issues (0 silenced).
E
======================================================================
ERROR: test_repr_partial_builtin_callable (urlpatterns_reverse.tests.ResolverMatchTests)
----------------------------------------------------------------------
Traceback (most recent call last):
  File "/testbed/tests/urlpatterns_reverse/tests.py", line 1151, in test_repr_partial_builtin_callable
    match = path('join/', view).resolve('join/')
  File "/testbed/django/urls/resolvers.py", line 378, in resolve
    return ResolverMatch(self.callback, args, kwargs, self.pattern.name, route=str(self.pattern))
  File "/testbed/django/urls/resolvers.py", line 49, in __init__
    self._func_path = self._get_func_path(func)
  File "/testbed/django/urls/resolvers.py", line 66, in _get_func_path
    return func.__module__ + '.' + func.__name__
AttributeError: 'method_descriptor' object has no attribute '__module__'

----------------------------------------------------------------------
Ran 1 test in 0.001s

FAILED (errors=1)
[pipeline] test_exit_code=1

```

## candidate_b
returncode=0 passed=True test_ran=True
log=02_execution/attempt_02/candidate_b.log
```
[pipeline] checking candidate patch
[pipeline] checking generated test patch
[pipeline] executing generated test command
Testing against Django installed in '/testbed/django' with up to 24 processes
System check identified no issues (0 silenced).
.
----------------------------------------------------------------------
Ran 1 test in 0.000s

OK
[pipeline] test_exit_code=0

```
</validated_execution>
