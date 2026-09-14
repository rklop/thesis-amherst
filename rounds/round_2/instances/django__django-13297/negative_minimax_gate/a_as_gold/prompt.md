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
diff --git a/django/views/generic/base.py b/django/views/generic/base.py
index ea5baca08d..e0307ec9f1 100644
--- a/django/views/generic/base.py
+++ b/django/views/generic/base.py
@@ -159,8 +159,8 @@ class TemplateView(TemplateResponseMixin, ContextMixin, View):
     def get(self, request, *args, **kwargs):
         # RemovedInDjango40Warning: when the deprecation ends, replace with:
         #   context = self.get_context_data()
-        context_kwargs = _wrap_url_kwargs_with_deprecation_warning(kwargs)
-        context = self.get_context_data(**context_kwargs)
+        context = self.get_context_data(**kwargs)
+        context.update(_wrap_url_kwargs_with_deprecation_warning(kwargs))
         return self.render_to_response(context)
 
 

</candidate_a_patch>
<candidate_b_patch>
diff --git a/django/views/generic/base.py b/django/views/generic/base.py
index ea5baca08d..6e0d677814 100644
--- a/django/views/generic/base.py
+++ b/django/views/generic/base.py
@@ -165,11 +165,20 @@ class TemplateView(TemplateResponseMixin, ContextMixin, View):
 
 
 # RemovedInDjango40Warning
+class _DeprecatedUrlKwargs(SimpleLazyObject):
+    """
+    A SimpleLazyObject that allows database-friendly access to the wrapped
+    value. Used to wrap URL kwargs passed to the context for deprecation
+    warning purposes.
+    """
+    def __conform__(self, protocol):
+        return self._wrapped
+
+
 def _wrap_url_kwargs_with_deprecation_warning(url_kwargs):
     context_kwargs = {}
     for key, value in url_kwargs.items():
         # Bind into function closure.
-        @SimpleLazyObject
         def access_value(key=key, value=value):
             warnings.warn(
                 'TemplateView passing URL kwargs to the context is '
@@ -178,7 +187,7 @@ def _wrap_url_kwargs_with_deprecation_warning(url_kwargs):
                 RemovedInDjango40Warning, stacklevel=2,
             )
             return value
-        context_kwargs[key] = access_value
+        context_kwargs[key] = _DeprecatedUrlKwargs(access_value)
     return context_kwargs
 
 

</candidate_b_patch>
<generated_test_proposal>
{
  "test_name": "test_get_context_data_url_kwargs_are_native_values",
  "specification_gap": "URL kwargs passed to an overridden TemplateView.get_context_data() must remain their resolver-produced native values. Deprecation-warning wrappers may be added to the final template context, but must not replace the values received by the override.",
  "input_description": "Invoke a TemplateView subclass with offer_slug='summer-sale'. Its get_context_data() JSON-serializes and deserializes the received kwargs, then exposes the result through response.context_data.",
  "expected_output": "response.context_data['received_url_kwargs'] is {'offer_slug': 'summer-sale'} without a JSON serialization error.",
  "expected_passing_candidate": "candidate_a",
  "why_interesting": "This exercises an ordinary non-database consumer of URL kwargs. Candidate B only adds database adaptation to the lazy wrapper, so a native string still reaches get_context_data() as a non-JSON-serializable wrapper; candidate A preserves the native value at that public override boundary.",
  "test_patch": "diff --git a/tests/generic_views/test_base.py b/tests/generic_views/test_base.py\n--- a/tests/generic_views/test_base.py\n+++ b/tests/generic_views/test_base.py\n@@ -1,3 +1,4 @@\n+import json\n import time\n \n from django.core.exceptions import ImproperlyConfigured\n@@ -308,9 +309,24 @@ class TemplateViewTest(SimpleTestCase):\n     def test_head(self):\n         \"\"\"\n         Test a TemplateView responds correctly to HEAD\n         \"\"\"\n         response = AboutTemplateView.as_view()(self.rf.head('/about/'))\n         self.assertEqual(response.status_code, 200)\n \n+    def test_get_context_data_url_kwargs_are_native_values(self):\n+        class JSONTemplateView(TemplateView):\n+            template_name = 'generic_views/about.html'\n+\n+            def get_context_data(self, **kwargs):\n+                return {'received_url_kwargs': json.loads(json.dumps(kwargs))}\n+\n+        response = JSONTemplateView.as_view()(\n+            self.rf.get('/'), offer_slug='summer-sale',\n+        )\n+        self.assertEqual(\n+            response.context_data['received_url_kwargs'],\n+            {'offer_slug': 'summer-sale'},\n+        )\n+\n     def test_get_template_attribute(self):\n         \"\"\"\n         Test a view that renders a template on GET with the template name as\n",
  "test_command": "cd /testbed && ./tests/runtests.py generic_views.test_base.TemplateViewTest.test_get_context_data_url_kwargs_are_native_values"
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
      "duration_seconds": 1.429,
      "log_path": "02_execution/attempt_01/candidate_a.log"
    },
    {
      "label": "candidate_b",
      "returncode": 1,
      "passed": false,
      "test_ran": true,
      "duration_seconds": 1.393,
      "log_path": "02_execution/attempt_01/candidate_b.log"
    }
  ]
}
</candidate_execution_summary>
<candidate_a_execution_log>
[pipeline] checking candidate patch
[pipeline] checking generated test patch
[pipeline] executing generated test command
.
----------------------------------------------------------------------
Ran 1 test in 0.001s

OK
Testing against Django installed in '/testbed/django' with up to 24 processes
System check identified no issues (0 silenced).
[pipeline] test_exit_code=0

</candidate_a_execution_log>
<candidate_b_execution_log>
[pipeline] checking candidate patch
[pipeline] checking generated test patch
[pipeline] executing generated test command
E
======================================================================
ERROR: test_get_context_data_url_kwargs_are_native_values (generic_views.test_base.TemplateViewTest)
----------------------------------------------------------------------
Traceback (most recent call last):
  File "/testbed/tests/generic_views/test_base.py", line 324, in test_get_context_data_url_kwargs_are_native_values
    self.rf.get('/'), offer_slug='summer-sale',
  File "/testbed/django/views/generic/base.py", line 73, in view
    return self.dispatch(request, *args, **kwargs)
  File "/testbed/django/views/generic/base.py", line 101, in dispatch
    return handler(request, *args, **kwargs)
  File "/testbed/django/views/generic/base.py", line 163, in get
    context = self.get_context_data(**context_kwargs)
  File "/testbed/tests/generic_views/test_base.py", line 321, in get_context_data
    return {'received_url_kwargs': json.loads(json.dumps(kwargs))}
  File "/opt/miniconda3/envs/testbed/lib/python3.6/json/__init__.py", line 231, in dumps
    return _default_encoder.encode(obj)
  File "/opt/miniconda3/envs/testbed/lib/python3.6/json/encoder.py", line 199, in encode
    chunks = self.iterencode(o, _one_shot=True)
  File "/opt/miniconda3/envs/testbed/lib/python3.6/json/encoder.py", line 257, in iterencode
    return _iterencode(o, 0)
  File "/opt/miniconda3/envs/testbed/lib/python3.6/json/encoder.py", line 180, in default
    o.__class__.__name__)
  File "/testbed/django/utils/functional.py", line 240, in inner
    self._setup()
  File "/testbed/django/utils/functional.py", line 376, in _setup
    self._wrapped = self._setupfunc()
  File "/testbed/django/views/generic/base.py", line 187, in access_value
    RemovedInDjango40Warning, stacklevel=2,
django.utils.deprecation.RemovedInDjango40Warning: TemplateView passing URL kwargs to the context is deprecated. Reference offer_slug in your template through view.kwargs instead.

----------------------------------------------------------------------
Ran 1 test in 0.001s

FAILED (errors=1)
Testing against Django installed in '/testbed/django' with up to 24 processes
System check identified no issues (0 silenced).
[pipeline] test_exit_code=1

</candidate_b_execution_log>
<official_gold_patch>
diff --git a/django/views/generic/base.py b/django/views/generic/base.py
--- a/django/views/generic/base.py
+++ b/django/views/generic/base.py
@@ -11,7 +11,7 @@
 from django.urls import reverse
 from django.utils.decorators import classonlymethod
 from django.utils.deprecation import RemovedInDjango40Warning
-from django.utils.functional import SimpleLazyObject
+from django.utils.functional import lazy
 
 logger = logging.getLogger('django.request')
 
@@ -169,7 +169,6 @@ def _wrap_url_kwargs_with_deprecation_warning(url_kwargs):
     context_kwargs = {}
     for key, value in url_kwargs.items():
         # Bind into function closure.
-        @SimpleLazyObject
         def access_value(key=key, value=value):
             warnings.warn(
                 'TemplateView passing URL kwargs to the context is '
@@ -178,7 +177,7 @@ def access_value(key=key, value=value):
                 RemovedInDjango40Warning, stacklevel=2,
             )
             return value
-        context_kwargs[key] = access_value
+        context_kwargs[key] = lazy(access_value, type(value))()
     return context_kwargs
 
 

</official_gold_patch>
<gold_execution_summary>
{
  "label": "gold",
  "returncode": 1,
  "passed": false,
  "test_ran": true,
  "duration_seconds": 1.416,
  "log_path": "gold_execution/gold.log"
}
</gold_execution_summary>
<gold_failure_contract>
{
  "kind": "behavioral_test_failure",
  "required_substrings": [
    "test_get_context_data_url_kwargs_are_native_values"
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
  "reference_log_sha256": "b4e8af01b35ab9df2bed4cd43fb21d30524bfe9474cf46903357c3062166eb0f"
}
</gold_failure_contract>
<gold_execution_log>
[pipeline] checking candidate patch
[pipeline] checking generated test patch
[pipeline] executing generated test command
ETesting against Django installed in '/testbed/django' with up to 24 processes
System check identified no issues (0 silenced).

======================================================================
ERROR: test_get_context_data_url_kwargs_are_native_values (generic_views.test_base.TemplateViewTest)
----------------------------------------------------------------------
Traceback (most recent call last):
  File "/testbed/tests/generic_views/test_base.py", line 324, in test_get_context_data_url_kwargs_are_native_values
    self.rf.get('/'), offer_slug='summer-sale',
  File "/testbed/django/views/generic/base.py", line 73, in view
    return self.dispatch(request, *args, **kwargs)
  File "/testbed/django/views/generic/base.py", line 101, in dispatch
    return handler(request, *args, **kwargs)
  File "/testbed/django/views/generic/base.py", line 163, in get
    context = self.get_context_data(**context_kwargs)
  File "/testbed/tests/generic_views/test_base.py", line 321, in get_context_data
    return {'received_url_kwargs': json.loads(json.dumps(kwargs))}
  File "/opt/miniconda3/envs/testbed/lib/python3.6/json/__init__.py", line 231, in dumps
    return _default_encoder.encode(obj)
  File "/opt/miniconda3/envs/testbed/lib/python3.6/json/encoder.py", line 199, in encode
    chunks = self.iterencode(o, _one_shot=True)
  File "/opt/miniconda3/envs/testbed/lib/python3.6/json/encoder.py", line 257, in iterencode
    return _iterencode(o, 0)
  File "/opt/miniconda3/envs/testbed/lib/python3.6/json/encoder.py", line 180, in default
    o.__class__.__name__)
TypeError: Object of type '__proxy__' is not JSON serializable

----------------------------------------------------------------------
Ran 1 test in 0.001s

FAILED (errors=1)
[pipeline] test_exit_code=1

</gold_execution_log>
