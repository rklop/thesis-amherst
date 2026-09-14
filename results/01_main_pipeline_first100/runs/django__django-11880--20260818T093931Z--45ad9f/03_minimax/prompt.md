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
Form Field’s __deepcopy__ does not (deep)copy the error messages.
Description
	
The __deepcopy__ method defined for the formfields (​https://github.com/django/django/blob/146086f219d01dbb1cd8c089b5a5667e396e1cc4/django/forms/fields.py#L200) performs a shallow copy of self and does not include additional treatment for the error_messages dictionary. As a result, all copies of the same field share the same dictionary and any modification of either the dictionary or the error message itself for one formfield is immediately reflected on all other formfiels.
This is relevant for Forms and ModelForms that modify the error messages of their fields dynamically: while each instance of the specific form (e.g., ProfileForm) is expected to have a set of fields “sealed” away from other instances of the same ProfileForm (​https://github.com/django/django/blob/146086f219d01dbb1cd8c089b5a5667e396e1cc4/django/forms/forms.py#L95), in fact all these instances share the same error messages, resulting in incorrectly raised errors.
Confirmed for versions of Django going back to 1.11.

</issue_statement>

<original_test_patch>
diff --git a/tests/forms_tests/tests/test_forms.py b/tests/forms_tests/tests/test_forms.py
--- a/tests/forms_tests/tests/test_forms.py
+++ b/tests/forms_tests/tests/test_forms.py
@@ -3685,6 +3685,17 @@ def test_empty_data_files_multi_value_dict(self):
         self.assertIsInstance(p.data, MultiValueDict)
         self.assertIsInstance(p.files, MultiValueDict)
 
+    def test_field_deep_copy_error_messages(self):
+        class CustomCharField(CharField):
+            def __init__(self, **kwargs):
+                kwargs['error_messages'] = {'invalid': 'Form custom error message.'}
+                super().__init__(**kwargs)
+
+        field = CustomCharField()
+        field_copy = copy.deepcopy(field)
+        self.assertIsInstance(field_copy, CustomCharField)
+        self.assertIsNot(field_copy.error_messages, field.error_messages)
+
 
 class CustomRenderer(DjangoTemplates):
     pass

</original_test_patch>

<candidate_a_patch>
diff --git a/django/forms/fields.py b/django/forms/fields.py
index 03cd8af8d7..18007accab 100644
--- a/django/forms/fields.py
+++ b/django/forms/fields.py
@@ -199,6 +199,7 @@ class Field:
         result = copy.copy(self)
         memo[id(self)] = result
         result.widget = copy.deepcopy(self.widget, memo)
+        result.error_messages = copy.deepcopy(self.error_messages, memo)
         result.validators = self.validators[:]
         return result
 

</candidate_a_patch>

<candidate_b_patch>
diff --git a/django/forms/fields.py b/django/forms/fields.py
--- a/django/forms/fields.py
+++ b/django/forms/fields.py
@@ -199,6 +199,7 @@ def __deepcopy__(self, memo):
         result = copy.copy(self)
         memo[id(self)] = result
         result.widget = copy.deepcopy(self.widget, memo)
+        result.error_messages = self.error_messages.copy()
         result.validators = self.validators[:]
         return result
 

</candidate_b_patch>

<generated_test_proposal>
{
  "test_name": "test_error_message_values_independence",
  "specification_gap": "The existing test requires only a distinct error_messages mapping, so a shallow dictionary copy passes. It doesn't specify whether mutable message values accepted by Django's ValidationError path are also isolated between form instances.",
  "input_description": "Define a Form whose required-field error message is a two-item list, instantiate two bound forms, and append a third message through only the first form's field.",
  "expected_output": "The first form reports ['First error.', 'Second error.', 'Third error.'], while the second still reports ['First error.', 'Second error.'].",
  "expected_passing_candidate": "candidate_a",
  "why_interesting": "This exercises the public Form construction and validation paths with a list-valued error message, which ValidationError supports. Candidate A recursively copies the list; candidate B copies only the surrounding dictionary and therefore leaks the appended message into the second form.",
  "test_patch": "diff --git a/tests/forms_tests/tests/test_forms.py b/tests/forms_tests/tests/test_forms.py\n--- a/tests/forms_tests/tests/test_forms.py\n+++ b/tests/forms_tests/tests/test_forms.py\n@@ -1398,6 +1398,19 @@ class FormsTestCase(SimpleTestCase):\n \n         f1.fields['myfield'].validators[0] = MaxValueValidator(12)\n         self.assertNotEqual(f1.fields['myfield'].validators[0], f2.fields['myfield'].validators[0])\n \n+    def test_error_message_values_independence(self):\n+        class MyForm(Form):\n+            name = CharField(error_messages={\n+                'required': ['First error.', 'Second error.'],\n+            })\n+\n+        f1 = MyForm({})\n+        f2 = MyForm({})\n+\n+        f1.fields['name'].error_messages['required'].append('Third error.')\n+        self.assertEqual(f1.errors['name'], ['First error.', 'Second error.', 'Third error.'])\n+        self.assertEqual(f2.errors['name'], ['First error.', 'Second error.'])\n+\n     def test_hidden_widget(self):\n         # HiddenInput widgets are displayed differently in the as_table(), as_ul())\n         # and as_p() output of a Form -- their verbose names are not displayed, and a\n",
  "test_command": "PYTHONPATH=. ./tests/runtests.py forms_tests.tests.test_forms.FormsTestCase.test_error_message_values_independence --verbosity 0"
}
</generated_test_proposal>

<validated_execution>
## candidate_a
returncode=0 passed=True test_ran=True
log=02_execution/attempt_01/candidate_a.log
```
[pipeline] checking candidate patch
[pipeline] checking generated test patch
[pipeline] executing generated test command
System check identified no issues (0 silenced).
----------------------------------------------------------------------
Ran 1 test in 0.000s

OK
[pipeline] test_exit_code=0

```

## candidate_b
returncode=1 passed=False test_ran=True
log=02_execution/attempt_01/candidate_b.log
```
[pipeline] checking candidate patch
[pipeline] checking generated test patch
[pipeline] executing generated test command
System check identified no issues (0 silenced).
======================================================================
FAIL: test_error_message_values_independence (forms_tests.tests.test_forms.FormsTestCase)
----------------------------------------------------------------------
Traceback (most recent call last):
  File "/testbed/tests/forms_tests/tests/test_forms.py", line 1413, in test_error_message_values_independence
    self.assertEqual(f2.errors['name'], ['First error.', 'Second error.'])
AssertionError: ['First error.', 'Second error.', 'Third error.'] != ['First error.', 'Second error.']

----------------------------------------------------------------------
Ran 1 test in 0.001s

FAILED (failures=1)
[pipeline] test_exit_code=1

```
</validated_execution>
