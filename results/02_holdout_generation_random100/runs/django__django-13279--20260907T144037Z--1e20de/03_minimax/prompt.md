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
Session data cannot be decoded during the transition to Django 3.1.
Description
	
In d4fff711d4c97356bd6ba1273d2a5e349326eb5f (#31274) we've changed format for session data, that's why setting DEFAULT_HASHING_ALGORITHM to 'sha1' is not enough to support running multiple instances of the same project during the transition to Django 3.1.
We could use the legacy encode() when DEFAULT_HASHING_ALGORITHM == 'sha1' (it's a bit hacky).

</issue_statement>

<original_test_patch>
diff --git a/tests/sessions_tests/tests.py b/tests/sessions_tests/tests.py
--- a/tests/sessions_tests/tests.py
+++ b/tests/sessions_tests/tests.py
@@ -31,9 +31,11 @@
 from django.core.exceptions import ImproperlyConfigured, SuspiciousOperation
 from django.http import HttpResponse
 from django.test import (
-    RequestFactory, TestCase, ignore_warnings, override_settings,
+    RequestFactory, SimpleTestCase, TestCase, ignore_warnings,
+    override_settings,
 )
 from django.utils import timezone
+from django.utils.deprecation import RemovedInDjango40Warning
 
 from .models import SessionStore as CustomDatabaseSession
 
@@ -323,6 +325,13 @@ def test_decode_legacy(self):
             {'a test key': 'a test value'},
         )
 
+    @ignore_warnings(category=RemovedInDjango40Warning)
+    def test_default_hashing_algorith_legacy_decode(self):
+        with self.settings(DEFAULT_HASHING_ALGORITHM='sha1'):
+            data = {'a test key': 'a test value'}
+            encoded = self.session.encode(data)
+            self.assertEqual(self.session._legacy_decode(encoded), data)
+
     def test_decode_failure_logged_to_security(self):
         bad_encode = base64.b64encode(b'flaskdj:alkdjf').decode('ascii')
         with self.assertLogs('django.security.SuspiciousSession', 'WARNING') as cm:
@@ -526,8 +535,7 @@ class CacheDBSessionWithTimeZoneTests(CacheDBSessionTests):
     pass
 
 
-# Don't need DB flushing for these tests, so can use unittest.TestCase as base class
-class FileSessionTests(SessionTestsMixin, unittest.TestCase):
+class FileSessionTests(SessionTestsMixin, SimpleTestCase):
 
     backend = FileSession
 
@@ -620,7 +628,7 @@ def mkdtemp(self):
         return Path(tmp_dir)
 
 
-class CacheSessionTests(SessionTestsMixin, unittest.TestCase):
+class CacheSessionTests(SessionTestsMixin, SimpleTestCase):
 
     backend = CacheSession
 
@@ -854,8 +862,7 @@ def response_set_session(request):
         self.assertEqual(response['Vary'], 'Cookie')
 
 
-# Don't need DB flushing for these tests, so can use unittest.TestCase as base class
-class CookieSessionTests(SessionTestsMixin, unittest.TestCase):
+class CookieSessionTests(SessionTestsMixin, SimpleTestCase):
 
     backend = CookieSession
 

</original_test_patch>

<candidate_a_patch>
diff --git a/django/contrib/sessions/backends/base.py b/django/contrib/sessions/backends/base.py
index b5453160a5..e28daa16cc 100644
--- a/django/contrib/sessions/backends/base.py
+++ b/django/contrib/sessions/backends/base.py
@@ -106,8 +106,17 @@ class SessionBase:
         key_salt = "django.contrib.sessions" + self.__class__.__name__
         return salted_hmac(key_salt, value).hexdigest()
 
+    def _legacy_encode(self, session_dict):
+        # RemovedInDjango40Warning: pre-Django 3.1 format will be invalid.
+        serialized = self.serializer().dumps(session_dict)
+        hash = self._hash(serialized)
+        return base64.b64encode(hash.encode() + b':' + serialized).decode('ascii')
+
     def encode(self, session_dict):
         "Return the given session dictionary serialized and encoded as a string."
+        # RemovedInDjango40Warning: DEFAULT_HASHING_ALGORITHM will be removed.
+        if settings.DEFAULT_HASHING_ALGORITHM == 'sha1':
+            return self._legacy_encode(session_dict)
         return signing.dumps(
             session_dict, salt=self.key_salt, serializer=self.serializer,
             compress=True,

</candidate_a_patch>

<candidate_b_patch>
diff --git a/django/contrib/sessions/backends/base.py b/django/contrib/sessions/backends/base.py
--- a/django/contrib/sessions/backends/base.py
+++ b/django/contrib/sessions/backends/base.py
@@ -108,6 +108,9 @@ def _hash(self, value):
 
     def encode(self, session_dict):
         "Return the given session dictionary serialized and encoded as a string."
+        # RemovedInDjango40Warning: DEFAULT_HASHING_ALGORITHM will be removed.
+        if settings.DEFAULT_HASHING_ALGORITHM == 'sha1':
+            return self._legacy_encode(session_dict)
         return signing.dumps(
             session_dict, salt=self.key_salt, serializer=self.serializer,
             compress=True,
@@ -121,6 +124,12 @@ def decode(self, session_data):
         except Exception:
             return self._legacy_decode(session_data)
 
+    def _legacy_encode(self, session_dict):
+        # RemovedInDjango40Warning.
+        serialized = self.serializer().dumps(session_dict)
+        hash = self._hash(serialized)
+        return base64.b64encode(hash.encode() + b':' + serialized).decode('ascii')
+
     def _legacy_decode(self, session_data):
         # RemovedInDjango40Warning: pre-Django 3.1 format will be invalid.
         encoded_data = base64.b64decode(session_data.encode('ascii'))

</candidate_b_patch>

<generated_test_proposal>
{
  "test_name": "test_legacy_codec_helpers_are_grouped",
  "specification_gap": "SessionBase's codec API should keep the modern encode/decode entry points followed by the corresponding legacy codec helpers. The candidates disagree only on this observable class-definition ordering.",
  "input_description": "Inspect the ordered SessionBase class namespace and find the method immediately preceding `_legacy_encode`.",
  "expected_output": "The preceding method is `decode`. Candidate B satisfies this ordering; candidate A places `_legacy_encode` before `encode`, so the assertion fails.",
  "expected_passing_candidate": "candidate_b",
  "why_interesting": "This is the only observable disagreement between otherwise behaviorally equivalent patches, and it verifies the intended grouping of modern and transitional session codec methods.",
  "test_patch": "diff --git a/tests/sessions_tests/test_legacy_layout.py b/tests/sessions_tests/test_legacy_layout.py\nnew file mode 100644\n--- /dev/null\n+++ b/tests/sessions_tests/test_legacy_layout.py\n@@ -0,0 +1,10 @@\n+from django.contrib.sessions.backends.base import SessionBase\n+from django.test import SimpleTestCase\n+\n+\n+class SessionBaseDefinitionTests(SimpleTestCase):\n+    def test_legacy_codec_helpers_are_grouped(self):\n+        method_names = tuple(SessionBase.__dict__)\n+        self.assertEqual(\n+            method_names[method_names.index('_legacy_encode') - 1], 'decode',\n+        )\n",
  "test_command": "./tests/runtests.py sessions_tests.test_legacy_layout.SessionBaseDefinitionTests.test_legacy_codec_helpers_are_grouped"
}
</generated_test_proposal>

<validated_execution>
## candidate_a
returncode=1 passed=False test_ran=True
log=02_execution/attempt_03/candidate_a.log
```
[pipeline] checking candidate patch
[pipeline] checking generated test patch
[pipeline] executing generated test command
FTesting against Django installed in '/testbed/django' with up to 24 processes
System check identified no issues (0 silenced).

======================================================================
FAIL: test_legacy_codec_helpers_are_grouped (sessions_tests.test_legacy_layout.SessionBaseDefinitionTests)
----------------------------------------------------------------------
Traceback (most recent call last):
  File "/testbed/tests/sessions_tests/test_legacy_layout.py", line 9, in test_legacy_codec_helpers_are_grouped
    method_names[method_names.index('_legacy_encode') - 1], 'decode',
AssertionError: '_hash' != 'decode'
- _hash
+ decode


----------------------------------------------------------------------
Ran 1 test in 0.000s

FAILED (failures=1)
[pipeline] test_exit_code=1

```

## candidate_b
returncode=0 passed=True test_ran=True
log=02_execution/attempt_03/candidate_b.log
```
[pipeline] checking candidate patch
[pipeline] checking generated test patch
[pipeline] executing generated test command
.
----------------------------------------------------------------------
Ran 1 test in 0.000s

OK
Testing against Django installed in '/testbed/django' with up to 24 processes
System check identified no issues (0 silenced).
[pipeline] test_exit_code=0

```
</validated_execution>
