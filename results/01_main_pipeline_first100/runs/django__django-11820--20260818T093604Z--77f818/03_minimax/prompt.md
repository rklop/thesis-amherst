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
models.E015 is raised when Meta.ordering contains "pk" of a related field.
Description
	
models.E015 is raised when Meta.ordering contains __pk of a related field, e.g.:
test_app.SomeModel: (models.E015) 'ordering' refers to the nonexistent field, related field, or lookup 'option__pk'.
Regression in 440505cb2cadbe1a5b9fba246bcde6c04f51d07e.

</issue_statement>

<original_test_patch>
diff --git a/tests/invalid_models_tests/test_models.py b/tests/invalid_models_tests/test_models.py
--- a/tests/invalid_models_tests/test_models.py
+++ b/tests/invalid_models_tests/test_models.py
@@ -814,6 +814,26 @@ class Meta:
             )
         ])
 
+    def test_ordering_pointing_multiple_times_to_model_fields(self):
+        class Parent(models.Model):
+            field1 = models.CharField(max_length=100)
+            field2 = models.CharField(max_length=100)
+
+        class Child(models.Model):
+            parent = models.ForeignKey(Parent, models.CASCADE)
+
+            class Meta:
+                ordering = ('parent__field1__field2',)
+
+        self.assertEqual(Child.check(), [
+            Error(
+                "'ordering' refers to the nonexistent field, related field, "
+                "or lookup 'parent__field1__field2'.",
+                obj=Child,
+                id='models.E015',
+            )
+        ])
+
     def test_ordering_allows_registered_lookups(self):
         class Model(models.Model):
             test = models.CharField(max_length=100)
@@ -824,6 +844,18 @@ class Meta:
         with register_lookup(models.CharField, Lower):
             self.assertEqual(Model.check(), [])
 
+    def test_ordering_pointing_to_related_model_pk(self):
+        class Parent(models.Model):
+            pass
+
+        class Child(models.Model):
+            parent = models.ForeignKey(Parent, models.CASCADE)
+
+            class Meta:
+                ordering = ('parent__pk',)
+
+        self.assertEqual(Child.check(), [])
+
     def test_ordering_pointing_to_foreignkey_field(self):
         class Parent(models.Model):
             pass

</original_test_patch>

<candidate_a_patch>
diff --git a/django/db/models/base.py b/django/db/models/base.py
index 3ce711f3f1..34505de068 100644
--- a/django/db/models/base.py
+++ b/django/db/models/base.py
@@ -1706,12 +1706,30 @@ class Model(metaclass=ModelBase):
         for field in related_fields:
             _cls = cls
             fld = None
-            for part in field.split(LOOKUP_SEP):
+            parts = field.split(LOOKUP_SEP)
+            for i, part in enumerate(parts):
                 try:
                     fld = _cls._meta.get_field(part)
                     if fld.is_relation:
                         _cls = fld.get_path_info()[-1].to_opts.model
+                    else:
+                        # Cannot traverse further through a non-relation field.
+                        if i < len(parts) - 1:
+                            # There are more parts to check - verify there's a valid transform.
+                            if fld.get_transform(parts[i + 1]) is None:
+                                errors.append(
+                                    checks.Error(
+                                        "'ordering' refers to the nonexistent field, "
+                                        "related field, or lookup '%s'." % field,
+                                        obj=cls,
+                                        id='models.E015',
+                                    )
+                                )
+                        break
                 except (FieldDoesNotExist, AttributeError):
+                    if part == 'pk':
+                        # 'pk' is an alias for the primary key, which is always valid.
+                        continue
                     if fld is None or fld.get_transform(part) is None:
                         errors.append(
                             checks.Error(

</candidate_a_patch>

<candidate_b_patch>
diff --git a/django/db/models/base.py b/django/db/models/base.py
--- a/django/db/models/base.py
+++ b/django/db/models/base.py
@@ -1708,9 +1708,15 @@ def _check_ordering(cls):
             fld = None
             for part in field.split(LOOKUP_SEP):
                 try:
-                    fld = _cls._meta.get_field(part)
+                    # pk is an alias that won't be found by opts.get_field.
+                    if part == 'pk':
+                        fld = _cls._meta.pk
+                    else:
+                        fld = _cls._meta.get_field(part)
                     if fld.is_relation:
                         _cls = fld.get_path_info()[-1].to_opts.model
+                    else:
+                        _cls = None
                 except (FieldDoesNotExist, AttributeError):
                     if fld is None or fld.get_transform(part) is None:
                         errors.append(

</candidate_b_patch>

<generated_test_proposal>
{
  "test_name": "test_ordering_rejects_lookup_after_registered_transform",
  "specification_gap": "Meta.ordering validation must inspect the entire lookup path. Recognizing a registered transform must not cause validation to ignore later nonexistent components.",
  "input_description": "Define a model with a CharField and Meta.ordering = ('test__lower__missing',), while registering the public Lower transform on CharField.",
  "expected_output": "Model.check() returns exactly one models.E015 Error naming 'test__lower__missing'. The 'lower' prefix is valid, but the trailing 'missing' lookup is not.",
  "expected_passing_candidate": "candidate_b",
  "why_interesting": "Candidate A stops checking after the first valid transform and therefore accepts any remaining suffix. Candidate B continues through all path components and rejects the nonexistent suffix. This tests the general invariant behind ordering-path validation without asserting implementation details.",
  "test_patch": "diff --git a/tests/invalid_models_tests/test_models.py b/tests/invalid_models_tests/test_models.py\n--- a/tests/invalid_models_tests/test_models.py\n+++ b/tests/invalid_models_tests/test_models.py\n@@ -821,9 +821,26 @@ class OtherModelTests(SimpleTestCase):\n             class Meta:\n                 ordering = ('test__lower',)\n \n         with register_lookup(models.CharField, Lower):\n             self.assertEqual(Model.check(), [])\n \n+    def test_ordering_rejects_lookup_after_registered_transform(self):\n+        class Model(models.Model):\n+            test = models.CharField(max_length=100)\n+\n+            class Meta:\n+                ordering = ('test__lower__missing',)\n+\n+        with register_lookup(models.CharField, Lower):\n+            self.assertEqual(Model.check(), [\n+                Error(\n+                    \"'ordering' refers to the nonexistent field, related field, \"\n+                    \"or lookup 'test__lower__missing'.\",\n+                    obj=Model,\n+                    id='models.E015',\n+                )\n+            ])\n+\n     def test_ordering_pointing_to_foreignkey_field(self):\n         class Parent(models.Model):\n             pass\n",
  "test_command": "cd /testbed && python tests/runtests.py invalid_models_tests.test_models.OtherModelTests.test_ordering_rejects_lookup_after_registered_transform"
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
FAIL: test_ordering_rejects_lookup_after_registered_transform (invalid_models_tests.test_models.OtherModelTests)
----------------------------------------------------------------------
Traceback (most recent call last):
  File "/testbed/tests/invalid_models_tests/test_models.py", line 840, in test_ordering_rejects_lookup_after_registered_transform
    id='models.E015',
AssertionError: Lists differ: [] != [<Error: level=40, msg="'ordering' refers [236 chars]15'>]

Second list contains 1 additional elements.
First extra element 0:
<Error: level=40, msg="'ordering' refers to the nonexistent field, related field, or lookup 'test__lower__missing'.", hint=None, obj=<class 'invalid_models_tests.test_models.OtherModelTests.test_ordering_rejects_lookup_after_registered_transform.<locals>.Model'>, id='models.E015'>

- []
+ [<Error: level=40, msg="'ordering' refers to the nonexistent field, related field, or lookup 'test__lower__missing'.", hint=None, obj=<class 'invalid_models_tests.test_models.OtherModelTests.test_ordering_rejects_lookup_after_registered_transform.<locals>.Model'>, id='models.E015'>]

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
