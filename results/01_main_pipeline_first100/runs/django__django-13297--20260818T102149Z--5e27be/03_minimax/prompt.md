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
TemplateView.get_context_data()'s kwargs returns SimpleLazyObjects that causes a crash when filtering.
Description
	
Example Code that works in 3.0, but not in 3.1:
class OfferView(TemplateView):
	template_name = "offers/offer.html"
	def get_context_data(self, **kwargs):
		offer_slug = kwargs.get("offer_slug", "")
		offer = get_object_or_404(Account, slug=offer_slug)
		return {"offer": offer, "offer_slug": offer_slug}
In order to make this work in 3.1, you have to explicitly convert the result of kwargs.get() to a string to get the SimpleLazyObject to resolve:
class OfferView(TemplateView):
	template_name = "offers/offer.html"
	def get_context_data(self, **kwargs):
		offer_slug = kwargs.get("offer_slug", "")
		offer = get_object_or_404(Account, slug=str(offer_slug))
		return {"offer": offer, "offer_slug": offer_slug}
The error generated if you don't is:
Error binding parameter 0 - probably unsupported type
from django/db/backends/sqlite3/operations.py, line 144, in _quote_params_for_last_executed_query
In both cases, the urls.py looks like:
path(
		"/offers/<slug:offer_slug>/",
		OfferView.as_view(),
		name="offer_view",
	),
When debugging, I found that offer_slug (coming in from kwargs.get) was of type 'SimpleLazyObject' in Django 3.1, and when I explicitly converted it to a string, get_object_or_404 behaved as expected.
This is using Python 3.7.8 with SQLite.

</issue_statement>

<original_test_patch>
diff --git a/tests/generic_views/test_base.py b/tests/generic_views/test_base.py
--- a/tests/generic_views/test_base.py
+++ b/tests/generic_views/test_base.py
@@ -3,7 +3,8 @@
 from django.core.exceptions import ImproperlyConfigured
 from django.http import HttpResponse
 from django.test import (
-    RequestFactory, SimpleTestCase, ignore_warnings, override_settings,
+    RequestFactory, SimpleTestCase, TestCase, ignore_warnings,
+    override_settings,
 )
 from django.test.utils import require_jinja2
 from django.urls import resolve
@@ -11,6 +12,7 @@
 from django.views.generic import RedirectView, TemplateView, View
 
 from . import views
+from .models import Artist
 
 
 class SimpleView(View):
@@ -571,7 +573,9 @@ def test_template_mixin_without_template(self):
 
 
 @override_settings(ROOT_URLCONF='generic_views.urls')
-class DeprecationTests(SimpleTestCase):
+class DeprecationTests(TestCase):
+    rf = RequestFactory()
+
     @ignore_warnings(category=RemovedInDjango40Warning)
     def test_template_params(self):
         """A generic template view passes kwargs as context."""
@@ -603,3 +607,17 @@ def test_template_params_warning(self):
             str(response.context['foo2'])
         self.assertEqual(response.context['key'], 'value')
         self.assertIsInstance(response.context['view'], View)
+
+    @ignore_warnings(category=RemovedInDjango40Warning)
+    def test_template_params_filtering(self):
+        class ArtistView(TemplateView):
+            template_name = 'generic_views/about.html'
+
+            def get_context_data(self, *, artist_name, **kwargs):
+                context = super().get_context_data(**kwargs)
+                artist = Artist.objects.get(name=artist_name)
+                return {**context, 'artist': artist}
+
+        artist = Artist.objects.create(name='Rene Magritte')
+        response = ArtistView.as_view()(self.rf.get('/'), artist_name=artist.name)
+        self.assertEqual(response.context_data['artist'], artist)

</original_test_patch>

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
 
 

</candidate_b_patch>

<generated_test_proposal>
{
  "test_name": "TemplateViewTest.test_extra_context_overrides_url_kwargs",
  "specification_gap": "When a deprecated URL keyword and TemplateView.extra_context use the same key, extra_context must retain ContextMixin's public merge precedence. Candidate A incorrectly reapplies URL kwargs after get_context_data(), overwriting extra_context; candidate B preserves the precedence.",
  "input_description": "Invoke TemplateView through as_view() with URL kwarg foo='from-url' and extra_context={'foo': 'from-extra-context'}.",
  "expected_output": "The returned TemplateResponse exposes response.context_data['foo'] == 'from-extra-context'.",
  "expected_passing_candidate": "candidate_b",
  "why_interesting": "This exercises a documented public entry point and a key-collision boundary. It detects a general context-composition regression without asserting proxy implementation details. The unified diff was validated with git apply --check against the pristine repository.",
  "test_patch": "diff --git a/tests/generic_views/test_base.py b/tests/generic_views/test_base.py\n--- a/tests/generic_views/test_base.py\n+++ b/tests/generic_views/test_base.py\n@@ -387,7 +387,18 @@ class TemplateViewTest(SimpleTestCase):\n \n     def test_extra_context(self):\n         response = self.client.get('/template/extra_context/')\n         self.assertEqual(response.context['title'], 'Title')\n+\n+    @ignore_warnings(category=RemovedInDjango40Warning)\n+    def test_extra_context_overrides_url_kwargs(self):\n+        view = TemplateView.as_view(\n+            template_name='generic_views/about.html',\n+            extra_context={'foo': 'from-extra-context'},\n+        )\n+        response = view(self.rf.get('/'), foo='from-url')\n+        self.assertEqual(\n+            response.context_data['foo'], 'from-extra-context',\n+        )\n \n \n @override_settings(ROOT_URLCONF='generic_views.urls')\n",
  "test_command": "./tests/runtests.py generic_views.test_base.TemplateViewTest.test_extra_context_overrides_url_kwargs"
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
FTesting against Django installed in '/testbed/django' with up to 24 processes
System check identified no issues (0 silenced).

======================================================================
FAIL: test_extra_context_overrides_url_kwargs (generic_views.test_base.TemplateViewTest)
----------------------------------------------------------------------
Traceback (most recent call last):
  File "/testbed/django/test/utils.py", line 381, in inner
    return func(*args, **kwargs)
  File "/testbed/tests/generic_views/test_base.py", line 400, in test_extra_context_overrides_url_kwargs
    response.context_data['foo'], 'from-extra-context',
AssertionError: <SimpleLazyObject: 'from-url'> != 'from-extra-context'

----------------------------------------------------------------------
Ran 1 test in 0.001s

FAILED (failures=1)
[pipeline] test_exit_code=1

```

## candidate_b
returncode=0 passed=True test_ran=True
log=02_execution/attempt_02/candidate_b.log
```
[pipeline] checking candidate patch
[pipeline] checking generated test patch
[pipeline] executing generated test command
.Testing against Django installed in '/testbed/django' with up to 24 processes
System check identified no issues (0 silenced).

----------------------------------------------------------------------
Ran 1 test in 0.001s

OK
[pipeline] test_exit_code=0

```
</validated_execution>
