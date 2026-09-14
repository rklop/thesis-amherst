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
Admin doesn't display properly unicode chars in JSONFields.
Description
	 
		(last modified by ZhaoQi99)
	 
>>> import json
>>> print json.dumps('中国')
"\u4e2d\u56fd"
json.dumps use ASCII encoding by default when serializing Chinese.
So when we edit a JsonField which contains Chinese character in Django admin,it will appear in ASCII characters.
I have try to fix this this problem in ​https://github.com/adamchainz/django-mysql/pull/714.And it works prefectly.

</issue_statement>

<original_test_patch>
diff --git a/tests/admin_utils/tests.py b/tests/admin_utils/tests.py
--- a/tests/admin_utils/tests.py
+++ b/tests/admin_utils/tests.py
@@ -186,6 +186,7 @@ def test_json_display_for_field(self):
             ({'a': {'b': 'c'}}, '{"a": {"b": "c"}}'),
             (['a', 'b'], '["a", "b"]'),
             ('a', '"a"'),
+            ({'a': '你好 世界'}, '{"a": "你好 世界"}'),
             ({('a', 'b'): 'c'}, "{('a', 'b'): 'c'}"),  # Invalid JSON.
         ]
         for value, display_value in tests:
diff --git a/tests/forms_tests/field_tests/test_jsonfield.py b/tests/forms_tests/field_tests/test_jsonfield.py
--- a/tests/forms_tests/field_tests/test_jsonfield.py
+++ b/tests/forms_tests/field_tests/test_jsonfield.py
@@ -29,6 +29,12 @@ def test_prepare_value(self):
         self.assertEqual(field.prepare_value({'a': 'b'}), '{"a": "b"}')
         self.assertEqual(field.prepare_value(None), 'null')
         self.assertEqual(field.prepare_value('foo'), '"foo"')
+        self.assertEqual(field.prepare_value('你好，世界'), '"你好，世界"')
+        self.assertEqual(field.prepare_value({'a': '😀🐱'}), '{"a": "😀🐱"}')
+        self.assertEqual(
+            field.prepare_value(["你好，世界", "jaźń"]),
+            '["你好，世界", "jaźń"]',
+        )
 
     def test_widget(self):
         field = JSONField()

</original_test_patch>

<candidate_a_patch>
diff --git a/django/db/models/fields/json.py b/django/db/models/fields/json.py
index a249f4cdbf..66c039c6e4 100644
--- a/django/db/models/fields/json.py
+++ b/django/db/models/fields/json.py
@@ -81,7 +81,7 @@ class JSONField(CheckFieldDefaultMixin, Field):
     def get_prep_value(self, value):
         if value is None:
             return value
-        return json.dumps(value, cls=self.encoder)
+        return json.dumps(value, cls=self.encoder, ensure_ascii=False)
 
     def get_transform(self, name):
         transform = super().get_transform(name)
diff --git a/django/forms/fields.py b/django/forms/fields.py
index 9de2c60b35..8235aaac01 100644
--- a/django/forms/fields.py
+++ b/django/forms/fields.py
@@ -1258,7 +1258,7 @@ class JSONField(CharField):
     def prepare_value(self, value):
         if isinstance(value, InvalidJSONInput):
             return value
-        return json.dumps(value, cls=self.encoder)
+        return json.dumps(value, cls=self.encoder, ensure_ascii=False)
 
     def has_changed(self, initial, data):
         if super().has_changed(initial, data):

</candidate_a_patch>

<candidate_b_patch>
diff --git a/django/contrib/admin/utils.py b/django/contrib/admin/utils.py
--- a/django/contrib/admin/utils.py
+++ b/django/contrib/admin/utils.py
@@ -1,5 +1,6 @@
 import datetime
 import decimal
+import json
 from collections import defaultdict
 
 from django.core.exceptions import FieldDoesNotExist
@@ -400,7 +401,7 @@ def display_for_field(value, field, empty_value_display):
         return format_html('<a href="{}">{}</a>', value.url, value)
     elif isinstance(field, models.JSONField) and value:
         try:
-            return field.get_prep_value(value)
+            return json.dumps(value, ensure_ascii=False, cls=field.encoder)
         except TypeError:
             return display_for_value(value, empty_value_display)
     else:
diff --git a/django/forms/fields.py b/django/forms/fields.py
--- a/django/forms/fields.py
+++ b/django/forms/fields.py
@@ -1258,7 +1258,7 @@ def bound_data(self, data, initial):
     def prepare_value(self, value):
         if isinstance(value, InvalidJSONInput):
             return value
-        return json.dumps(value, cls=self.encoder)
+        return json.dumps(value, ensure_ascii=False, cls=self.encoder)
 
     def has_changed(self, initial, data):
         if super().has_changed(initial, data):

</candidate_b_patch>

<generated_test_proposal>
{
  "test_name": "test_json_display_preserves_database_serialization",
  "specification_gap": "Non-ASCII JSON should be readable in admin presentation without changing JSONField's database-preparation behavior. Candidate A changes serialization globally, while candidate B confines the change to presentation entry points.",
  "input_description": "Pass {'country': '\u4e2d\u56fd'} through admin display_for_field() and through get_prep_value() on the same models.JSONField instance.",
  "expected_output": "display_for_field() returns '{\"country\": \"\u4e2d\u56fd\"}', while get_prep_value() retains the database serialization '{\"country\": \"\\u4e2d\\u56fd\"}'.",
  "expected_passing_candidate": "candidate_b",
  "why_interesting": "This exercises two public entry points and verifies the issue's stated scope: improve admin display without affecting database writes. The patch was validated with git apply --check.",
  "test_patch": "diff --git a/tests/admin_utils/tests.py b/tests/admin_utils/tests.py\n--- a/tests/admin_utils/tests.py\n+++ b/tests/admin_utils/tests.py\n@@ -195,6 +195,15 @@ class UtilsTests(SimpleTestCase):\n                     display_value,\n                 )\n \n+    def test_json_display_preserves_database_serialization(self):\n+        field = models.JSONField()\n+        value = {'country': '\u4e2d\u56fd'}\n+        self.assertEqual(\n+            display_for_field(value, field, self.empty_value),\n+            '{\"country\": \"\u4e2d\u56fd\"}',\n+        )\n+        self.assertEqual(field.get_prep_value(value), '{\"country\": \"\\\\u4e2d\\\\u56fd\"}')\n+\n     def test_number_formats_display_for_field(self):\n         display_value = display_for_field(12345.6789, models.FloatField(), self.empty_value)\n         self.assertEqual(display_value, '12345.6789')\n",
  "test_command": "python tests/runtests.py admin_utils.tests.UtilsTests.test_json_display_preserves_database_serialization"
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
F
======================================================================
FAIL: test_json_display_preserves_database_serialization (admin_utils.tests.UtilsTests)
----------------------------------------------------------------------
Traceback (most recent call last):
  File "/testbed/tests/admin_utils/tests.py", line 205, in test_json_display_preserves_database_serialization
    self.assertEqual(field.get_prep_value(value), '{"country": "\\u4e2d\\u56fd"}')
AssertionError: '{"country": "\u4e2d\u56fd"}' != '{"country": "\\u4e2d\\u56fd"}'
- {"country": "\u4e2d\u56fd"}
+ {"country": "\u4e2d\u56fd"}


----------------------------------------------------------------------
Ran 1 test in 0.000s

FAILED (failures=1)
Testing against Django installed in '/testbed/django' with up to 24 processes
System check identified no issues (0 silenced).
[pipeline] test_exit_code=1

```

## candidate_b
returncode=0 passed=True test_ran=True
log=02_execution/attempt_02/candidate_b.log
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
