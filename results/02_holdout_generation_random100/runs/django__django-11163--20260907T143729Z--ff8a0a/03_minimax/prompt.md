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
model_to_dict() should return an empty dict for an empty list of fields.
Description
	
Been called as model_to_dict(instance, fields=[]) function should return empty dict, because no fields were requested. But it returns all fields
The problem point is
if fields and f.name not in fields:
which should be
if fields is not None and f.name not in fields:
PR: ​https://github.com/django/django/pull/11150/files

</issue_statement>

<original_test_patch>
diff --git a/tests/model_forms/tests.py b/tests/model_forms/tests.py
--- a/tests/model_forms/tests.py
+++ b/tests/model_forms/tests.py
@@ -1814,6 +1814,10 @@ class Meta:
 
         bw = BetterWriter.objects.create(name='Joe Better', score=10)
         self.assertEqual(sorted(model_to_dict(bw)), ['id', 'name', 'score', 'writer_ptr'])
+        self.assertEqual(sorted(model_to_dict(bw, fields=[])), [])
+        self.assertEqual(sorted(model_to_dict(bw, fields=['id', 'name'])), ['id', 'name'])
+        self.assertEqual(sorted(model_to_dict(bw, exclude=[])), ['id', 'name', 'score', 'writer_ptr'])
+        self.assertEqual(sorted(model_to_dict(bw, exclude=['id', 'name'])), ['score', 'writer_ptr'])
 
         form = BetterWriterForm({'name': 'Some Name', 'score': 12})
         self.assertTrue(form.is_valid())

</original_test_patch>

<candidate_a_patch>
diff --git a/django/forms/models.py b/django/forms/models.py
index d157c291ef..fadac848d0 100644
--- a/django/forms/models.py
+++ b/django/forms/models.py
@@ -83,7 +83,7 @@ def model_to_dict(instance, fields=None, exclude=None):
     for f in chain(opts.concrete_fields, opts.private_fields, opts.many_to_many):
         if not getattr(f, 'editable', False):
             continue
-        if fields and f.name not in fields:
+        if fields is not None and f.name not in fields:
             continue
         if exclude and f.name in exclude:
             continue
@@ -431,7 +431,7 @@ class BaseModelForm(BaseForm):
         for f in chain(opts.many_to_many, opts.private_fields):
             if not hasattr(f, 'save_form_data'):
                 continue
-            if fields and f.name not in fields:
+            if fields is not None and f.name not in fields:
                 continue
             if exclude and f.name in exclude:
                 continue

</candidate_a_patch>

<candidate_b_patch>
diff --git a/django/forms/models.py b/django/forms/models.py
--- a/django/forms/models.py
+++ b/django/forms/models.py
@@ -83,7 +83,7 @@ def model_to_dict(instance, fields=None, exclude=None):
     for f in chain(opts.concrete_fields, opts.private_fields, opts.many_to_many):
         if not getattr(f, 'editable', False):
             continue
-        if fields and f.name not in fields:
+        if fields is not None and f.name not in fields:
             continue
         if exclude and f.name in exclude:
             continue

</candidate_b_patch>

<generated_test_proposal>
{
  "test_name": "test_declared_many_to_many_field_saved_with_empty_fields",
  "specification_gap": "A declaratively defined ModelForm field remains part of the form even when Meta.fields is an empty list. The specification does not explicitly state whether save() must still persist such a declared many-to-many field. The established public behavior is that the validated declared field is saved.",
  "input_description": "Create a ModelForm for ColourfulItem with an explicitly declared colours ModelMultipleChoiceField and Meta.fields = []. Bind it to an existing item with one selected Colour, validate it, and call save().",
  "expected_output": "The saved item's colours relation contains the submitted Colour. Candidate B preserves this behavior; candidate A's additional empty-fields check silently skips the many-to-many update, leaving the relation empty.",
  "expected_passing_candidate": "candidate_b",
  "why_interesting": "This exercises ModelForm.save(), an alternate public entry point beyond model_to_dict(). It detects candidate A's unrelated behavioral expansion while relying on Django's intentional retention of explicitly declared fields when Meta.fields is empty.",
  "test_patch": "diff --git a/tests/model_forms/tests.py b/tests/model_forms/tests.py\n--- a/tests/model_forms/tests.py\n+++ b/tests/model_forms/tests.py\n@@ -351,13 +351,28 @@ class ModelFormBaseTest(TestCase):\n     def test_replace_field_variant_3(self):\n         # Should have the same result as before,\n         # but 'fields' attribute specified differently\n         class ReplaceField(forms.ModelForm):\n             url = forms.BooleanField()\n \n             class Meta:\n                 model = Category\n                 fields = []  # url will still appear, since it is explicit above\n \n         self.assertIsInstance(ReplaceField.base_fields['url'], forms.fields.BooleanField)\n \n+    def test_declared_many_to_many_field_saved_with_empty_fields(self):\n+        class ColourForm(forms.ModelForm):\n+            colours = forms.ModelMultipleChoiceField(queryset=Colour.objects.all())\n+\n+            class Meta:\n+                model = ColourfulItem\n+                fields = []\n+\n+        colour = Colour.objects.create(name='blue')\n+        item = ColourfulItem.objects.create(name='item')\n+        form = ColourForm({'colours': [colour.pk]}, instance=item)\n+        self.assertTrue(form.is_valid())\n+        form.save()\n+        self.assertEqual(list(item.colours.all()), [colour])\n+\n     def test_override_field(self):\n",
  "test_command": "python tests/runtests.py model_forms.tests.ModelFormBaseTest.test_declared_many_to_many_field_saved_with_empty_fields"
}
</generated_test_proposal>

<validated_execution>
## candidate_a
returncode=1 passed=False test_ran=True
log=02_execution/attempt_01/candidate_a.log
```
[pipeline] checking candidate patch
[pipeline] checking generated test patch
[pipeline] executing generated test command
Creating test database for alias 'default'...
F
======================================================================
FAIL: test_declared_many_to_many_field_saved_with_empty_fields (model_forms.tests.ModelFormBaseTest)
----------------------------------------------------------------------
Traceback (most recent call last):
  File "/testbed/tests/model_forms/tests.py", line 376, in test_declared_many_to_many_field_saved_with_empty_fields
    self.assertEqual(list(item.colours.all()), [colour])
AssertionError: Lists differ: [] != [<Colour: blue>]

Second list contains 1 additional elements.
First extra element 0:
<Colour: blue>

- []
+ [<Colour: blue>]

----------------------------------------------------------------------
Ran 1 test in 0.002s

FAILED (failures=1)
Destroying test database for alias 'default'...
Testing against Django installed in '/testbed/django' with up to 24 processes
System check identified no issues (0 silenced).
[pipeline] test_exit_code=1

```

## candidate_b
returncode=0 passed=True test_ran=True
log=02_execution/attempt_01/candidate_b.log
```
[pipeline] checking candidate patch
[pipeline] checking generated test patch
[pipeline] executing generated test command
Creating test database for alias 'default'...
.
----------------------------------------------------------------------
Ran 1 test in 0.002s

OK
Destroying test database for alias 'default'...
Testing against Django installed in '/testbed/django' with up to 24 processes
System check identified no issues (0 silenced).
[pipeline] test_exit_code=0

```
</validated_execution>
