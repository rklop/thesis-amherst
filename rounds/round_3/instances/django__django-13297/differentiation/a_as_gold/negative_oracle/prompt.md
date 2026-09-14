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
<candidate_a_patch>
diff --git a/django/db/backends/sqlite3/base.py b/django/db/backends/sqlite3/base.py
index 8a105d4f35..3fb9385697 100644
--- a/django/db/backends/sqlite3/base.py
+++ b/django/db/backends/sqlite3/base.py
@@ -24,6 +24,7 @@ from django.utils.asyncio import async_unsafe
 from django.utils.dateparse import parse_datetime, parse_time
 from django.utils.duration import duration_microseconds
 from django.utils.regex_helper import _lazy_re_compile
+from django.utils.functional import SimpleLazyObject
 from django.utils.version import PY38
 
 from .client import DatabaseClient                          # isort:skip
@@ -75,6 +76,7 @@ Database.register_converter("datetime", decoder(parse_datetime))
 Database.register_converter("timestamp", decoder(parse_datetime))
 
 Database.register_adapter(decimal.Decimal, str)
+Database.register_adapter(SimpleLazyObject, str)
 
 
 class DatabaseWrapper(BaseDatabaseWrapper):

</candidate_a_patch>
<candidate_b_patch>
diff --git a/django/utils/functional.py b/django/utils/functional.py
index 6d38f932f9..a6a4ed05a2 100644
--- a/django/utils/functional.py
+++ b/django/utils/functional.py
@@ -321,6 +321,7 @@ class LazyObject:
         return copy.deepcopy(self._wrapped, memo)
 
     __bytes__ = new_method_proxy(bytes)
+    __conform__ = new_method_proxy(lambda wrapped, protocol: wrapped)
     __str__ = new_method_proxy(str)
     __bool__ = new_method_proxy(bool)
 

</candidate_b_patch>
<generated_test_proposal>
{
  "test_name": "test_simple_lazy_object_with_uuid_parameter",
  "specification_gap": "SQLite must accept a SimpleLazyObject as a query parameter by adapting its string representation, even when the wrapped value is not itself a SQLite-native scalar. This covers UUID-valued URL kwargs in addition to the reported slug-string case.",
  "input_description": "Execute SELECT %s through Django's SQLite cursor with a SimpleLazyObject wrapping UUID('550e8400-e29b-41d4-a716-446655440000').",
  "expected_output": "The query succeeds and fetchone()[0] is the string '550e8400-e29b-41d4-a716-446655440000'.",
  "expected_passing_candidate": "candidate_a",
  "why_interesting": "Both patches handle a lazy string, but UUID is a standard URL-converter output and a useful type boundary. candidate_a string-adapts the lazy object; candidate_b exposes the wrapped UUID to SQLite's binding protocol, where it remains an unsupported parameter type.",
  "test_patch": "diff --git a/tests/backends/sqlite/tests.py b/tests/backends/sqlite/tests.py\n--- a/tests/backends/sqlite/tests.py\n+++ b/tests/backends/sqlite/tests.py\n@@ -2,6 +2,7 @@ import re\n import tempfile\n import threading\n import unittest\n+import uuid\n from pathlib import Path\n from sqlite3 import dbapi2\n from unittest import mock\n@@ -14,6 +15,7 @@ from django.test import (\n     TestCase, TransactionTestCase, override_settings, skipIfDBFeature,\n )\n from django.test.utils import isolate_apps\n+from django.utils.functional import SimpleLazyObject\n \n from ..models import Author, Item, Object, Square\n \n@@ -97,6 +99,13 @@ class Tests(TestCase):\n                     cursor.execute('SELECT %s REGEXP %s', [string, pattern])\n                     value = cursor.fetchone()[0]\n                 value = bool(value) if value in {0, 1} else value\n                 self.assertIs(value, expected)\n \n+    def test_simple_lazy_object_with_uuid_parameter(self):\n+        value = uuid.UUID('550e8400-e29b-41d4-a716-446655440000')\n+        lazy_value = SimpleLazyObject(lambda: value)\n+        with connection.cursor() as cursor:\n+            cursor.execute('SELECT %s', [lazy_value])\n+            self.assertEqual(cursor.fetchone()[0], str(value))\n+\n     def test_pathlib_name(self):\n         with tempfile.TemporaryDirectory() as tmp:\n             settings_dict = {\n",
  "test_command": "cd /testbed && ./tests/runtests.py backends.sqlite.tests.Tests.test_simple_lazy_object_with_uuid_parameter -v 2"
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
      "duration_seconds": 1.677,
      "log_path": "02_execution/attempt_01/candidate_a.log"
    },
    {
      "label": "candidate_b",
      "returncode": 1,
      "passed": false,
      "test_ran": true,
      "duration_seconds": 1.479,
      "log_path": "02_execution/attempt_01/candidate_b.log"
    }
  ]
}
</candidate_execution_summary>
<candidate_a_execution_log>
[pipeline] checking candidate patch
[pipeline] checking generated test patch
[pipeline] executing generated test command
Creating test database for alias 'default' ('file:memorydb_default?mode=memory&cache=shared')...
test_simple_lazy_object_with_uuid_parameter (backends.sqlite.tests.Tests) ... ok

----------------------------------------------------------------------
Ran 1 test in 0.001s

OK
Destroying test database for alias 'default' ('file:memorydb_default?mode=memory&cache=shared')...
Testing against Django installed in '/testbed/django' with up to 24 processes
Importing application backends
Skipping setup of unused database(s): other.
Operations to perform:
  Synchronize unmigrated apps: auth, backends, contenttypes, messages, sessions, staticfiles
  Apply all migrations: admin, sites
Synchronizing apps without migrations:
  Creating tables...
    Creating table django_content_type
    Creating table auth_permission
    Creating table auth_group
    Creating table auth_user
    Creating table django_session
    Creating table backends_square
    Creating table backends_person
    Creating table backends_schoolclass
    Creating table backends_verylongmodelnamezzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzz
    Creating table backends_tag
    Creating table CaseSensitive_Post
    Creating table backends_reporter
    Creating table backends_article
    Creating table backends_item
    Creating table backends_object
    Creating table backends_objectreference
    Creating table backends_objectselfreference
    Creating table backends_circulara
    Creating table backends_circularb
    Creating table backends_rawdata
    Creating table backends_author
    Creating table backends_book
    Running deferred SQL...
Running migrations:
  Applying admin.0001_initial... OK
  Applying admin.0002_logentry_remove_auto_add... OK
  Applying admin.0003_logentry_add_action_flag_choices... OK
  Applying sites.0001_initial... OK
  Applying sites.0002_alter_domain_unique... OK
System check identified no issues (0 silenced).
[pipeline] test_exit_code=0

</candidate_a_execution_log>
<candidate_b_execution_log>
[pipeline] checking candidate patch
[pipeline] checking generated test patch
[pipeline] executing generated test command
Creating test database for alias 'default' ('file:memorydb_default?mode=memory&cache=shared')...
test_simple_lazy_object_with_uuid_parameter (backends.sqlite.tests.Tests) ... ERROR

======================================================================
ERROR: test_simple_lazy_object_with_uuid_parameter (backends.sqlite.tests.Tests)
----------------------------------------------------------------------
Traceback (most recent call last):
  File "/testbed/django/db/backends/utils.py", line 84, in _execute
    return self.cursor.execute(sql, params)
  File "/testbed/django/db/backends/sqlite3/base.py", line 412, in execute
    return Database.Cursor.execute(self, query, params)
sqlite3.InterfaceError: Error binding parameter 0 - probably unsupported type.

The above exception was the direct cause of the following exception:

Traceback (most recent call last):
  File "/testbed/tests/backends/sqlite/tests.py", line 108, in test_simple_lazy_object_with_uuid_parameter
    cursor.execute('SELECT %s', [lazy_value])
  File "/testbed/django/db/backends/utils.py", line 66, in execute
    return self._execute_with_wrappers(sql, params, many=False, executor=self._execute)
  File "/testbed/django/db/backends/utils.py", line 75, in _execute_with_wrappers
    return executor(sql, params, many, context)
  File "/testbed/django/db/backends/utils.py", line 84, in _execute
    return self.cursor.execute(sql, params)
  File "/testbed/django/db/utils.py", line 90, in __exit__
    raise dj_exc_value.with_traceback(traceback) from exc_value
  File "/testbed/django/db/backends/utils.py", line 84, in _execute
    return self.cursor.execute(sql, params)
  File "/testbed/django/db/backends/sqlite3/base.py", line 412, in execute
    return Database.Cursor.execute(self, query, params)
django.db.utils.InterfaceError: Error binding parameter 0 - probably unsupported type.

----------------------------------------------------------------------
Ran 1 test in 0.001s

FAILED (errors=1)
Destroying test database for alias 'default' ('file:memorydb_default?mode=memory&cache=shared')...
Testing against Django installed in '/testbed/django' with up to 24 processes
Importing application backends
Skipping setup of unused database(s): other.
Operations to perform:
  Synchronize unmigrated apps: auth, backends, contenttypes, messages, sessions, staticfiles
  Apply all migrations: admin, sites
Synchronizing apps without migrations:
  Creating tables...
    Creating table django_content_type
    Creating table auth_permission
    Creating table auth_group
    Creating table auth_user
    Creating table django_session
    Creating table backends_square
    Creating table backends_person
    Creating table backends_schoolclass
    Creating table backends_verylongmodelnamezzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzz
    Creating table backends_tag
    Creating table CaseSensitive_Post
    Creating table backends_reporter
    Creating table backends_article
    Creating table backends_item
    Creating table backends_object
    Creating table backends_objectreference
    Creating table backends_objectselfreference
    Creating table backends_circulara
    Creating table backends_circularb
    Creating table backends_rawdata
    Creating table backends_author
    Creating table backends_book
    Running deferred SQL...
Running migrations:
  Applying admin.0001_initial... OK
  Applying admin.0002_logentry_remove_auto_add... OK
  Applying admin.0003_logentry_add_action_flag_choices... OK
  Applying sites.0001_initial... OK
  Applying sites.0002_alter_domain_unique... OK
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
  "duration_seconds": 1.524,
  "log_path": "gold_execution/gold.log"
}
</gold_execution_summary>
<gold_failure_contract>
{
  "kind": "behavioral_test_failure",
  "required_substrings": [
    "test_simple_lazy_object_with_uuid_parameter"
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
  "reference_log_sha256": "3436911211644012e2fa6b7845b7e5903ecf134bb90a66e7e0faa40d77c5fba8"
}
</gold_failure_contract>
<gold_execution_log>
[pipeline] checking candidate patch
[pipeline] checking generated test patch
[pipeline] executing generated test command
Creating test database for alias 'default' ('file:memorydb_default?mode=memory&cache=shared')...
test_simple_lazy_object_with_uuid_parameter (backends.sqlite.tests.Tests) ... ERROR

======================================================================
ERROR: test_simple_lazy_object_with_uuid_parameter (backends.sqlite.tests.Tests)
----------------------------------------------------------------------
Traceback (most recent call last):
  File "/testbed/django/db/backends/utils.py", line 84, in _execute
    return self.cursor.execute(sql, params)
  File "/testbed/django/db/backends/sqlite3/base.py", line 412, in execute
    return Database.Cursor.execute(self, query, params)
sqlite3.InterfaceError: Error binding parameter 0 - probably unsupported type.

The above exception was the direct cause of the following exception:

Traceback (most recent call last):
  File "/testbed/tests/backends/sqlite/tests.py", line 108, in test_simple_lazy_object_with_uuid_parameter
    cursor.execute('SELECT %s', [lazy_value])
  File "/testbed/django/db/backends/utils.py", line 66, in execute
    return self._execute_with_wrappers(sql, params, many=False, executor=self._execute)
  File "/testbed/django/db/backends/utils.py", line 75, in _execute_with_wrappers
    return executor(sql, params, many, context)
  File "/testbed/django/db/backends/utils.py", line 84, in _execute
    return self.cursor.execute(sql, params)
  File "/testbed/django/db/utils.py", line 90, in __exit__
    raise dj_exc_value.with_traceback(traceback) from exc_value
  File "/testbed/django/db/backends/utils.py", line 84, in _execute
    return self.cursor.execute(sql, params)
  File "/testbed/django/db/backends/sqlite3/base.py", line 412, in execute
    return Database.Cursor.execute(self, query, params)
django.db.utils.InterfaceError: Error binding parameter 0 - probably unsupported type.

----------------------------------------------------------------------
Ran 1 test in 0.001s

FAILED (errors=1)
Destroying test database for alias 'default' ('file:memorydb_default?mode=memory&cache=shared')...
Testing against Django installed in '/testbed/django' with up to 24 processes
Importing application backends
Skipping setup of unused database(s): other.
Operations to perform:
  Synchronize unmigrated apps: auth, backends, contenttypes, messages, sessions, staticfiles
  Apply all migrations: admin, sites
Synchronizing apps without migrations:
  Creating tables...
    Creating table django_content_type
    Creating table auth_permission
    Creating table auth_group
    Creating table auth_user
    Creating table django_session
    Creating table backends_square
    Creating table backends_person
    Creating table backends_schoolclass
    Creating table backends_verylongmodelnamezzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzz
    Creating table backends_tag
    Creating table CaseSensitive_Post
    Creating table backends_reporter
    Creating table backends_article
    Creating table backends_item
    Creating table backends_object
    Creating table backends_objectreference
    Creating table backends_objectselfreference
    Creating table backends_circulara
    Creating table backends_circularb
    Creating table backends_rawdata
    Creating table backends_author
    Creating table backends_book
    Running deferred SQL...
Running migrations:
  Applying admin.0001_initial... OK
  Applying admin.0002_logentry_remove_auto_add... OK
  Applying admin.0003_logentry_add_action_flag_choices... OK
  Applying sites.0001_initial... OK
  Applying sites.0002_alter_domain_unique... OK
System check identified no issues (0 silenced).
[pipeline] test_exit_code=1

</gold_execution_log>
