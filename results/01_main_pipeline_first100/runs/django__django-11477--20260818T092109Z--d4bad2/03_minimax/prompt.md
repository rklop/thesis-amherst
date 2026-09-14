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
translate_url() creates an incorrect URL when optional named groups are missing in the URL pattern
Description
	
There is a problem when translating urls with absent 'optional' arguments
(it's seen in test case of the patch)

</issue_statement>

<original_test_patch>
diff --git a/tests/i18n/patterns/tests.py b/tests/i18n/patterns/tests.py
--- a/tests/i18n/patterns/tests.py
+++ b/tests/i18n/patterns/tests.py
@@ -158,6 +158,15 @@ def test_translate_url_utility(self):
             # path() URL pattern
             self.assertEqual(translate_url('/en/account/register-as-path/', 'nl'), '/nl/profiel/registreren-als-pad/')
             self.assertEqual(translation.get_language(), 'en')
+            # URL with parameters.
+            self.assertEqual(
+                translate_url('/en/with-arguments/regular-argument/', 'nl'),
+                '/nl/with-arguments/regular-argument/',
+            )
+            self.assertEqual(
+                translate_url('/en/with-arguments/regular-argument/optional.html', 'nl'),
+                '/nl/with-arguments/regular-argument/optional.html',
+            )
 
         with translation.override('nl'):
             self.assertEqual(translate_url('/nl/gebruikers/', 'en'), '/en/users/')
diff --git a/tests/i18n/patterns/urls/default.py b/tests/i18n/patterns/urls/default.py
--- a/tests/i18n/patterns/urls/default.py
+++ b/tests/i18n/patterns/urls/default.py
@@ -15,6 +15,11 @@
 urlpatterns += i18n_patterns(
     path('prefixed/', view, name='prefixed'),
     path('prefixed.xml', view, name='prefixed_xml'),
+    re_path(
+        _(r'^with-arguments/(?P<argument>[\w-]+)/(?:(?P<optional>[\w-]+).html)?$'),
+        view,
+        name='with-arguments',
+    ),
     re_path(_(r'^users/$'), view, name='users'),
     re_path(_(r'^account/'), include('i18n.patterns.urls.namespace', namespace='account')),
 )
diff --git a/tests/urlpatterns/path_urls.py b/tests/urlpatterns/path_urls.py
--- a/tests/urlpatterns/path_urls.py
+++ b/tests/urlpatterns/path_urls.py
@@ -11,6 +11,7 @@
     path('users/<id>/', views.empty_view, name='user-with-id'),
     path('included_urls/', include('urlpatterns.included_urls')),
     re_path(r'^regex/(?P<pk>[0-9]+)/$', views.empty_view, name='regex'),
+    re_path(r'^regex_optional/(?P<arg1>\d+)/(?:(?P<arg2>\d+)/)?', views.empty_view, name='regex_optional'),
     path('', include('urlpatterns.more_urls')),
     path('<lang>/<path:url>/', views.empty_view, name='lang-and-path'),
 ]
diff --git a/tests/urlpatterns/tests.py b/tests/urlpatterns/tests.py
--- a/tests/urlpatterns/tests.py
+++ b/tests/urlpatterns/tests.py
@@ -54,6 +54,20 @@ def test_re_path(self):
         self.assertEqual(match.kwargs, {'pk': '1'})
         self.assertEqual(match.route, '^regex/(?P<pk>[0-9]+)/$')
 
+    def test_re_path_with_optional_parameter(self):
+        for url, kwargs in (
+            ('/regex_optional/1/2/', {'arg1': '1', 'arg2': '2'}),
+            ('/regex_optional/1/', {'arg1': '1'}),
+        ):
+            with self.subTest(url=url):
+                match = resolve(url)
+                self.assertEqual(match.url_name, 'regex_optional')
+                self.assertEqual(match.kwargs, kwargs)
+                self.assertEqual(
+                    match.route,
+                    r'^regex_optional/(?P<arg1>\d+)/(?:(?P<arg2>\d+)/)?',
+                )
+
     def test_path_lookup_with_inclusion(self):
         match = resolve('/included_urls/extra/something/')
         self.assertEqual(match.url_name, 'inner-extra')
diff --git a/tests/urlpatterns_reverse/tests.py b/tests/urlpatterns_reverse/tests.py
--- a/tests/urlpatterns_reverse/tests.py
+++ b/tests/urlpatterns_reverse/tests.py
@@ -180,6 +180,8 @@
     ('named_optional', '/optional/1/', [], {'arg1': 1}),
     ('named_optional', '/optional/1/2/', [1, 2], {}),
     ('named_optional', '/optional/1/2/', [], {'arg1': 1, 'arg2': 2}),
+    ('named_optional_terminated', '/optional/1/', [1], {}),
+    ('named_optional_terminated', '/optional/1/', [], {'arg1': 1}),
     ('named_optional_terminated', '/optional/1/2/', [1, 2], {}),
     ('named_optional_terminated', '/optional/1/2/', [], {'arg1': 1, 'arg2': 2}),
     ('hardcoded', '/hardcoded/', [], {}),

</original_test_patch>

<candidate_a_patch>
diff --git a/django/urls/resolvers.py b/django/urls/resolvers.py
index af0508f94e..7cbe3e7e45 100644
--- a/django/urls/resolvers.py
+++ b/django/urls/resolvers.py
@@ -153,7 +153,7 @@ class RegexPattern(CheckURLMixin):
             # If there are any named groups, use those as kwargs, ignoring
             # non-named groups. Otherwise, pass all non-named arguments as
             # positional arguments.
-            kwargs = match.groupdict()
+            kwargs = {k: v for k, v in match.groupdict().items() if v is not None}
             args = () if kwargs else match.groups()
             return path[match.end():], args, kwargs
         return None
@@ -249,7 +249,7 @@ class RoutePattern(CheckURLMixin):
         match = self.regex.search(path)
         if match:
             # RoutePattern doesn't allow non-named groups so args are ignored.
-            kwargs = match.groupdict()
+            kwargs = {k: v for k, v in match.groupdict().items() if v is not None}
             for key, value in kwargs.items():
                 converter = self.converters[key]
                 try:

</candidate_a_patch>

<candidate_b_patch>
diff --git a/django/urls/resolvers.py b/django/urls/resolvers.py
--- a/django/urls/resolvers.py
+++ b/django/urls/resolvers.py
@@ -153,7 +153,7 @@ def match(self, path):
             # If there are any named groups, use those as kwargs, ignoring
             # non-named groups. Otherwise, pass all non-named arguments as
             # positional arguments.
-            kwargs = match.groupdict()
+            kwargs = {k: v for k, v in match.groupdict().items() if v is not None}
             args = () if kwargs else match.groups()
             return path[match.end():], args, kwargs
         return None

</candidate_b_patch>

<generated_test_proposal>
{
  "test_name": "test_converter_ignores_unmatched_named_subgroups",
  "specification_gap": "Optional named regex subgroups should be omitted consistently when resolving path() routes, including subgroups inside a registered custom converter. Only parameters declared by the route should be converted and exposed as view kwargs.",
  "input_description": "Register a custom path converter whose regex matches either \"primary-<qualifier>\" or \"fallback\". Resolve \"/item/fallback/\", leaving the converter's internal named \"qualifier\" subgroup unmatched.",
  "expected_output": "resolve() succeeds with URL name \"optional-subgroup\" and kwargs {'value': 'FALLBACK'}. The unmatched internal subgroup is neither exposed nor treated as a route converter.",
  "expected_passing_candidate": "candidate_a",
  "why_interesting": "Candidate A filters unmatched groups in both RegexPattern and RoutePattern, while candidate B only changes RegexPattern. This tests the documented custom-converter path() entry point and an observable resolution invariant rather than re-testing re_path().",
  "test_patch": "diff --git a/tests/urlpatterns/path_optional_subgroup_urls.py b/tests/urlpatterns/path_optional_subgroup_urls.py\nnew file mode 100644\n--- /dev/null\n+++ b/tests/urlpatterns/path_optional_subgroup_urls.py\n@@ -0,0 +1,20 @@\n+from django.urls import path, register_converter\n+\n+from .views import empty_view\n+\n+\n+class OptionalSubgroupConverter:\n+    regex = r'(?:primary-(?P<qualifier>[a-z]+)|fallback)'\n+\n+    def to_python(self, value):\n+        return value.upper()\n+\n+    def to_url(self, value):\n+        return value\n+\n+\n+register_converter(OptionalSubgroupConverter, 'optional_subgroup')\n+\n+urlpatterns = [\n+    path('item/<optional_subgroup:value>/', empty_view, name='optional-subgroup'),\n+]\ndiff --git a/tests/urlpatterns/tests.py b/tests/urlpatterns/tests.py\n--- a/tests/urlpatterns/tests.py\n+++ b/tests/urlpatterns/tests.py\n@@ -84,6 +84,12 @@ class SimplifiedURLTests(SimpleTestCase):\n                 self.assertEqual(match.url_name, url_name)\n                 self.assertEqual(match.app_name, app_name)\n                 self.assertEqual(match.kwargs, kwargs)\n+\n+    @override_settings(ROOT_URLCONF='urlpatterns.path_optional_subgroup_urls')\n+    def test_converter_ignores_unmatched_named_subgroups(self):\n+        match = resolve('/item/fallback/')\n+        self.assertEqual(match.url_name, 'optional-subgroup')\n+        self.assertEqual(match.kwargs, {'value': 'FALLBACK'})\n \n     @override_settings(ROOT_URLCONF='urlpatterns.path_base64_urls')\n     def test_converter_reverse(self):\n",
  "test_command": "cd /testbed && python tests/runtests.py urlpatterns.tests.SimplifiedURLTests.test_converter_ignores_unmatched_named_subgroups"
}
</generated_test_proposal>

<validated_execution>
## candidate_a
returncode=0 passed=True test_ran=True
log=02_execution/attempt_02/candidate_a.log
```
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

```

## candidate_b
returncode=1 passed=False test_ran=True
log=02_execution/attempt_02/candidate_b.log
```
[pipeline] checking candidate patch
[pipeline] checking generated test patch
[pipeline] executing generated test command
E
======================================================================
ERROR: test_converter_ignores_unmatched_named_subgroups (urlpatterns.tests.SimplifiedURLTests)
----------------------------------------------------------------------
Traceback (most recent call last):
  File "/testbed/django/test/utils.py", line 370, in inner
    return func(*args, **kwargs)
  File "/testbed/tests/urlpatterns/tests.py", line 91, in test_converter_ignores_unmatched_named_subgroups
    match = resolve('/item/fallback/')
  File "/testbed/django/urls/base.py", line 25, in resolve
    return get_resolver(urlconf).resolve(path)
  File "/testbed/django/urls/resolvers.py", line 538, in resolve
    sub_match = pattern.resolve(new_path)
  File "/testbed/django/urls/resolvers.py", line 345, in resolve
    match = self.pattern.match(path)
  File "/testbed/django/urls/resolvers.py", line 254, in match
    converter = self.converters[key]
KeyError: 'qualifier'

----------------------------------------------------------------------
Ran 1 test in 0.001s

FAILED (errors=1)
Testing against Django installed in '/testbed/django' with up to 24 processes
System check identified no issues (0 silenced).
[pipeline] test_exit_code=1

```
</validated_execution>
