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
FieldError when migrating field to new model subclass.
Description
	
Analogous to #21890. If creating a model subclass and moving a field onto it in the same step, makemigrations works but migrate dies with django.core.exceptions.FieldError: Local field 'title' in class 'Book' clashes with field of the same name from base class 'Readable'.
For example, take this model:
from django.db import models
class Readable(models.Model):
	title = models.CharField(max_length=200)
And change to this:
from django.db import models
class Readable(models.Model):
	pass
class Book(Readable):
	title = models.CharField(max_length=200)
The migration generates with CreateModel for Book, then RemoveField for Readable.title. But running it produces the error.
Reversing the order of the migration operations makes it pass. The auto-detector should be able to use this order.

</issue_statement>

<original_test_patch>
diff --git a/tests/migrations/test_autodetector.py b/tests/migrations/test_autodetector.py
--- a/tests/migrations/test_autodetector.py
+++ b/tests/migrations/test_autodetector.py
@@ -2454,3 +2454,28 @@ def test_mti_inheritance_model_removal(self):
         self.assertNumberMigrations(changes, 'app', 1)
         self.assertOperationTypes(changes, 'app', 0, ['DeleteModel'])
         self.assertOperationAttributes(changes, 'app', 0, 0, name='Dog')
+
+    def test_add_model_with_field_removed_from_base_model(self):
+        """
+        Removing a base field takes place before adding a new inherited model
+        that has a field with the same name.
+        """
+        before = [
+            ModelState('app', 'readable', [
+                ('id', models.AutoField(primary_key=True)),
+                ('title', models.CharField(max_length=200)),
+            ]),
+        ]
+        after = [
+            ModelState('app', 'readable', [
+                ('id', models.AutoField(primary_key=True)),
+            ]),
+            ModelState('app', 'book', [
+                ('title', models.CharField(max_length=200)),
+            ], bases=('app.readable',)),
+        ]
+        changes = self.get_changes(before, after)
+        self.assertNumberMigrations(changes, 'app', 1)
+        self.assertOperationTypes(changes, 'app', 0, ['RemoveField', 'CreateModel'])
+        self.assertOperationAttributes(changes, 'app', 0, 0, name='title', model_name='readable')
+        self.assertOperationAttributes(changes, 'app', 0, 1, name='book')

</original_test_patch>

<candidate_a_patch>
diff --git a/django/db/migrations/autodetector.py b/django/db/migrations/autodetector.py
index 30616c0172..d133c01dd3 100644
--- a/django/db/migrations/autodetector.py
+++ b/django/db/migrations/autodetector.py
@@ -563,6 +563,20 @@ class MigrationAutodetector:
                 if isinstance(base, str) and "." in base:
                     base_app_label, base_name = base.split(".", 1)
                     dependencies.append((base_app_label, base_name, None, True))
+            # Depend on the removal of any fields from base classes that have
+            # the same name as fields on this model. This ensures that when
+            # moving a field from a base class to a subclass, the RemoveField
+            # operation happens before the CreateModel operation.
+            for base in model_state.bases:
+                if isinstance(base, str) and "." in base:
+                    base_app_label, base_name = base.split(".", 1)
+                    base_key = (base_app_label, base_name.lower())
+                    for field_name in model_state.fields:
+                        if base_key in self.kept_model_keys:
+                            old_fields = self.from_state.models.get(base_key)
+                            if old_fields and field_name in old_fields.fields:
+                                if (base_app_label, base_name.lower(), field_name) not in self.new_field_keys:
+                                    dependencies.append((base_app_label, base_name.lower(), field_name, False))
             # Depend on the other end of the primary key if it's a relation
             if primary_key_rel:
                 dependencies.append((
diff --git a/tests/migrations/test_autodetector.py b/tests/migrations/test_autodetector.py
index daff57fc2c..10af72561b 100644
--- a/tests/migrations/test_autodetector.py
+++ b/tests/migrations/test_autodetector.py
@@ -2454,3 +2454,28 @@ class AutodetectorTests(TestCase):
         self.assertNumberMigrations(changes, 'app', 1)
         self.assertOperationTypes(changes, 'app', 0, ['DeleteModel'])
         self.assertOperationAttributes(changes, 'app', 0, 0, name='Dog')
+
+    def test_add_model_with_field_removed_from_base_model(self):
+        """
+        Removing a base field takes place before adding a new inherited model
+        that has a field with the same name.
+        """
+        before = [
+            ModelState('app', 'readable', [
+                ('id', models.AutoField(primary_key=True)),
+                ('title', models.CharField(max_length=200)),
+            ]),
+        ]
+        after = [
+            ModelState('app', 'readable', [
+                ('id', models.AutoField(primary_key=True)),
+            ]),
+            ModelState('app', 'book', [
+                ('title', models.CharField(max_length=200)),
+            ], bases=('app.readable',)),
+        ]
+        changes = self.get_changes(before, after)
+        self.assertNumberMigrations(changes, 'app', 1)
+        self.assertOperationTypes(changes, 'app', 0, ['RemoveField', 'CreateModel'])
+        self.assertOperationAttributes(changes, 'app', 0, 0, name='title', model_name='readable')
+        self.assertOperationAttributes(changes, 'app', 0, 1, name='book')

</candidate_a_patch>

<candidate_b_patch>
diff --git a/django/db/migrations/autodetector.py b/django/db/migrations/autodetector.py
--- a/django/db/migrations/autodetector.py
+++ b/django/db/migrations/autodetector.py
@@ -563,6 +563,16 @@ def generate_created_models(self):
                 if isinstance(base, str) and "." in base:
                     base_app_label, base_name = base.split(".", 1)
                     dependencies.append((base_app_label, base_name, None, True))
+                    # Depend on the removal of base fields if the new model has
+                    # a field with the same name.
+                    old_base_model_state = self.from_state.models.get((base_app_label, base_name))
+                    new_base_model_state = self.to_state.models.get((base_app_label, base_name))
+                    if old_base_model_state and new_base_model_state:
+                        removed_base_fields = set(old_base_model_state.fields).difference(
+                            new_base_model_state.fields,
+                        ).intersection(model_state.fields)
+                        for removed_base_field in removed_base_fields:
+                            dependencies.append((base_app_label, base_name, removed_base_field, False))
             # Depend on the other end of the primary key if it's a relation
             if primary_key_rel:
                 dependencies.append((

</candidate_b_patch>

<generated_test_proposal>
{
  "test_name": "test_add_model_with_field_removed_from_mixed_case_base_model",
  "specification_gap": "Moving a field from a concrete base to a newly created subclass must order RemoveField before CreateModel even when the subclass's public string base reference uses the model's original mixed-case name. Django treats string model references case-insensitively, so dependency detection must do the same.",
  "input_description": "The old state contains Readable with a title field. The new state removes Readable.title and creates Book with its own title field, inheriting through the mixed-case string reference 'app.Readable'.",
  "expected_output": "The autodetector produces one app migration whose operations are exactly RemoveField followed by CreateModel, preventing the inherited-field clash.",
  "expected_passing_candidate": "candidate_a",
  "why_interesting": "Candidate A normalizes the base model name before looking it up in ProjectState; candidate B looks up the mixed-case name verbatim even though ProjectState keys are lowercase. This tests a general case-insensitivity invariant already exposed by Django's public string model references, rather than another copy of the lowercase benchmark example.",
  "test_patch": "diff --git a/tests/migrations/test_autodetector.py b/tests/migrations/test_autodetector.py\n--- a/tests/migrations/test_autodetector.py\n+++ b/tests/migrations/test_autodetector.py\n@@ -2443,9 +2443,32 @@ class AutodetectorTests(TestCase):\n         # Right number/type of migrations?\n         self.assertNumberMigrations(changes, 'testapp', 1)\n         self.assertOperationTypes(changes, 'testapp', 0, [\"AddField\", \"AddField\"])\n         self.assertOperationAttributes(changes, 'testapp', 0, 0)\n \n+    def test_add_model_with_field_removed_from_mixed_case_base_model(self):\n+        \"\"\"\n+        Model references are case-insensitive when ordering field removal\n+        before creation of an inheriting model.\n+        \"\"\"\n+        before = [\n+            ModelState('app', 'Readable', [\n+                ('id', models.AutoField(primary_key=True)),\n+                ('title', models.CharField(max_length=200)),\n+            ]),\n+        ]\n+        after = [\n+            ModelState('app', 'Readable', [\n+                ('id', models.AutoField(primary_key=True)),\n+            ]),\n+            ModelState('app', 'Book', [\n+                ('title', models.CharField(max_length=200)),\n+            ], bases=('app.Readable',)),\n+        ]\n+        changes = self.get_changes(before, after)\n+        self.assertNumberMigrations(changes, 'app', 1)\n+        self.assertOperationTypes(changes, 'app', 0, ['RemoveField', 'CreateModel'])\n+\n     def test_mti_inheritance_model_removal(self):\n         Animal = ModelState('app', 'Animal', [\n             (\"id\", models.AutoField(primary_key=True)),\n         ])\n",
  "test_command": "cd /testbed && ./tests/runtests.py migrations.test_autodetector.AutodetectorTests.test_add_model_with_field_removed_from_mixed_case_base_model"
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
Creating test database for alias 'default'...
.
----------------------------------------------------------------------
Ran 1 test in 0.001s

OK
Destroying test database for alias 'default'...
Testing against Django installed in '/testbed/django' with up to 24 processes
System check identified no issues (0 silenced).
[pipeline] test_exit_code=0

```

## candidate_b
returncode=1 passed=False test_ran=True
log=02_execution/attempt_01/candidate_b.log
```
[pipeline] checking candidate patch
[pipeline] checking generated test patch
[pipeline] executing generated test command
Creating test database for alias 'default'...
F
======================================================================
FAIL: test_add_model_with_field_removed_from_mixed_case_base_model (migrations.test_autodetector.AutodetectorTests)
----------------------------------------------------------------------
Traceback (most recent call last):
  File "/testbed/tests/migrations/test_autodetector.py", line 2469, in test_add_model_with_field_removed_from_mixed_case_base_model
    self.assertOperationTypes(changes, 'app', 0, ['RemoveField', 'CreateModel'])
  File "/testbed/tests/migrations/test_autodetector.py", line 517, in assertOperationTypes
    self.repr_changes(changes),
AssertionError: Operation type mismatch for app.auto_1 (expected ['RemoveField', 'CreateModel']):
  app:
    auto_1
      <CreateModel  name='Book', fields=[('title', <django.db.models.fields.CharField>)], options={}, bases=('app.Readable',), managers=[]>
      <RemoveField  model_name='readable', name='title'>


----------------------------------------------------------------------
Ran 1 test in 0.002s

FAILED (failures=1)
Destroying test database for alias 'default'...
Testing against Django installed in '/testbed/django' with up to 24 processes
System check identified no issues (0 silenced).
[pipeline] test_exit_code=1

```
</validated_execution>
