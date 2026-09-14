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
diff --git a/django/forms/models.py b/django/forms/models.py
index d157c291ef..5edbbd376f 100644
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

</candidate_a_patch>
<candidate_b_patch>
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

</candidate_b_patch>
<generated_test_proposal>
{
  "test_name": "test_empty_fields_do_not_save_declared_many_to_many_field",
  "specification_gap": "An empty ModelForm Meta.fields list means no model fields are selected for persistence, including many-to-many fields that are explicitly declared on the form and therefore appear in cleaned_data.",
  "input_description": "Create an item related to an existing colour, then bind it to a ModelForm with Meta.fields = [] and an explicitly declared colours field containing a different colour. Call the public form.save() method.",
  "expected_output": "The item's persisted colour relation remains the original colour; the submitted replacement is not saved because colours isn't selected by Meta.fields.",
  "expected_passing_candidate": "candidate_b",
  "why_interesting": "This exercises the empty-field-selection invariant through ModelForm relation saving. candidate_a fixes model_to_dict() but still treats [] as unrestricted in _save_m2m(), while candidate_b fixes both paths.",
  "test_patch": "diff --git a/tests/model_forms/tests.py b/tests/model_forms/tests.py\n--- a/tests/model_forms/tests.py\n+++ b/tests/model_forms/tests.py\n@@ -191,13 +191,39 @@ class ModelFormBaseTest(TestCase):\n     def test_empty_fields_on_modelform(self):\n         \"\"\"\n         No fields on a ModelForm should actually result in no fields.\n         \"\"\"\n         class EmptyPersonForm(forms.ModelForm):\n             class Meta:\n                 model = Person\n                 fields = ()\n \n         form = EmptyPersonForm()\n         self.assertEqual(len(form.fields), 0)\n \n+    def test_empty_fields_do_not_save_declared_many_to_many_field(self):\n+        \"\"\"\n+        ModelForm.save() doesn't save fields omitted by an empty Meta.fields.\n+        \"\"\"\n+        class EmptyItemForm(forms.ModelForm):\n+            colours = forms.ModelMultipleChoiceField(Colour.objects.all())\n+\n+            class Meta:\n+                model = ColourfulItem\n+                fields = []\n+\n+        old_colour = Colour.objects.create(name='Old')\n+        new_colour = Colour.objects.create(name='New')\n+        item = ColourfulItem.objects.create(name='Item')\n+        item.colours.add(old_colour)\n+\n+        form = EmptyItemForm(\n+            {'colours': [new_colour.pk]}, instance=item,\n+        )\n+        self.assertTrue(form.is_valid())\n+        form.save()\n+        self.assertEqual(\n+            list(item.colours.values_list('pk', flat=True)),\n+            [old_colour.pk],\n+        )\n+\n     def test_empty_fields_to_construct_instance(self):\n         \"\"\"\n         No fields should be set on a model instance if construct_instance receives fields=().\n",
  "test_command": "cd /testbed && ./tests/runtests.py model_forms.tests.ModelFormBaseTest.test_empty_fields_do_not_save_declared_many_to_many_field"
}
</generated_test_proposal>
<candidate_execution_summary>
{
  "attempt": 1,
  "separates": true,
  "required_passing_candidate": "candidate_b",
  "meets_acceptance_rule": true,
  "results": [
    {
      "label": "candidate_a",
      "returncode": 1,
      "passed": false,
      "test_ran": true,
      "duration_seconds": 1.689,
      "log_path": "02_execution/attempt_01/candidate_a.log"
    },
    {
      "label": "candidate_b",
      "returncode": 0,
      "passed": true,
      "test_ran": true,
      "duration_seconds": 1.735,
      "log_path": "02_execution/attempt_01/candidate_b.log"
    }
  ]
}
</candidate_execution_summary>
<candidate_a_execution_log>
[pipeline] checking candidate patch
[pipeline] checking generated test patch
[pipeline] executing generated test command
Creating test database for alias 'default'...
F
======================================================================
FAIL: test_empty_fields_do_not_save_declared_many_to_many_field (model_forms.tests.ModelFormBaseTest)
----------------------------------------------------------------------
Traceback (most recent call last):
  File "/testbed/tests/model_forms/tests.py", line 226, in test_empty_fields_do_not_save_declared_many_to_many_field
    [old_colour.pk],
AssertionError: Lists differ: [2] != [1]

First differing element 0:
2
1

- [2]
+ [1]

----------------------------------------------------------------------
Ran 1 test in 0.004s

FAILED (failures=1)
Destroying test database for alias 'default'...
Testing against Django installed in '/testbed/django' with up to 24 processes
System check identified no issues (0 silenced).
[pipeline] test_exit_code=1

</candidate_a_execution_log>
<candidate_b_execution_log>
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

</candidate_b_execution_log>
<official_gold_patch>
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

</official_gold_patch>
<gold_execution_summary>
{
  "label": "gold",
  "returncode": 1,
  "passed": false,
  "test_ran": true,
  "duration_seconds": 1.836,
  "log_path": "gold_execution/gold.log"
}
</gold_execution_summary>
<gold_failure_contract>
{
  "kind": "behavioral_test_failure",
  "required_substrings": [
    "test_empty_fields_do_not_save_declared_many_to_many_field"
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
  "reference_log_sha256": "fed0222f62764008e9a49a93cea2d3abb9c07575090044ebc7efd3144d12e9d0"
}
</gold_failure_contract>
<gold_execution_log>
[pipeline] checking candidate patch
[pipeline] checking generated test patch
[pipeline] executing generated test command
Creating test database for alias 'default'...
F
======================================================================
FAIL: test_empty_fields_do_not_save_declared_many_to_many_field (model_forms.tests.ModelFormBaseTest)
----------------------------------------------------------------------
Traceback (most recent call last):
  File "/testbed/tests/model_forms/tests.py", line 226, in test_empty_fields_do_not_save_declared_many_to_many_field
    [old_colour.pk],
AssertionError: Lists differ: [2] != [1]

First differing element 0:
2
1

- [2]
+ [1]

----------------------------------------------------------------------
Ran 1 test in 0.003s

FAILED (failures=1)
Destroying test database for alias 'default'...
Testing against Django installed in '/testbed/django' with up to 24 processes
System check identified no issues (0 silenced).
[pipeline] test_exit_code=1

</gold_execution_log>
