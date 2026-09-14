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
ModelAdmin.lookup_allowed() incorrectly raises DisallowedModelAdminLookup lookup with foreign key as primary key
Description
	 
		(last modified by Tim Graham)
	 
Wrote a failing test for tests/modeladmin/tests.py to demonstrate - same test/code passes on 1.8
@isolate_apps('modeladmin')
def test_lookup_allowed_foreign_primary(self):
	class Country(models.Model):
		name = models.CharField(max_length=256)
	class Place(models.Model):
		country = models.ForeignKey(Country, models.CASCADE)
	class Restaurant(models.Model):
		place = models.OneToOneField(Place, models.CASCADE, primary_key=True)
	class Waiter(models.Model):
		restaurant = models.ForeignKey(Restaurant, models.CASCADE)
	class WaiterAdmin(ModelAdmin):
		list_filter = [
			'restaurant__place__country',
		]
	ma = WaiterAdmin(Waiter, self.site)
	self.assertIs(ma.lookup_allowed('restaurant__place__country', 'test_value'), True)
I think this is caused by the admin thinking that having a foreign key field as a primary key is the same as concrete inheritance. So when you try and check lookups for restaurant__place__country it thinks 'place' is the concrete parent of 'restaurant' and shortcuts it to restaurant__country which isn't in 'list_filter'. And you can't add restaurant__country to list_filter because country isn't actually on restaurant.

</issue_statement>
<candidate_a_patch>
diff --git a/django/contrib/admin/options.py b/django/contrib/admin/options.py
index 5240b07cd6..e527d196d2 100644
--- a/django/contrib/admin/options.py
+++ b/django/contrib/admin/options.py
@@ -469,6 +469,14 @@ class BaseModelAdmin(metaclass=forms.MediaDefiningClass):
                 and field not in prev_field.path_infos[-1].target_fields
             ):
                 relation_parts.append(part)
+            elif (
+                prev_field
+                and prev_field.is_relation
+                and field in prev_field.path_infos[-1].target_fields
+                and hasattr(field, "path_infos")
+            ):
+                # Field is in target_fields but can be further traversed (e.g., OneToOneField as pk).
+                relation_parts.append(part)
             if not getattr(field, "path_infos", None):
                 # This is not a relational field, so further parts
                 # must be transforms.

</candidate_a_patch>
<candidate_b_patch>
diff --git a/django/contrib/admin/options.py b/django/contrib/admin/options.py
index 5240b07cd6..fd97d2dd35 100644
--- a/django/contrib/admin/options.py
+++ b/django/contrib/admin/options.py
@@ -466,7 +466,10 @@ class BaseModelAdmin(metaclass=forms.MediaDefiningClass):
             # then the id value would be found already from employee__department_id.
             if not prev_field or (
                 prev_field.is_relation
-                and field not in prev_field.path_infos[-1].target_fields
+                and (
+                    field.is_relation
+                    or field not in prev_field.path_infos[-1].target_fields
+                )
             ):
                 relation_parts.append(part)
             if not getattr(field, "path_infos", None):

</candidate_b_patch>
<generated_test_proposal>
{
  "test_name": "test_lookup_allowed_foreign_key_target_is_nonrelational",
  "specification_gap": "A ForeignKey target component must be treated as an additional relation only when the target field is actually relational. Merely exposing an attribute named path_infos must not change a scalar target lookup into a relational lookup.",
  "input_description": "Define a custom scalar IntegerField primary key with an empty path_infos attribute, point a ForeignKey at it, and call ModelAdmin.lookup_allowed(\"target__code\", \"1\") with no list_filter. The lookup addresses the related primary-key value already available through the local ForeignKey column.",
  "expected_output": "lookup_allowed() returns True. The supplied gold checks field.is_relation and preserves the local-value behavior; the generated candidate mistakes the attribute's presence for relationality and returns False.",
  "expected_passing_candidate": "candidate_b",
  "why_interesting": "This tests the public custom Field extension point and the existing invariant that a ForeignKey target value is locally available. It distinguishes semantic relation classification from the candidate's structural attribute check without inspecting internal state.",
  "test_patch": "diff --git a/tests/modeladmin/tests.py b/tests/modeladmin/tests.py\n--- a/tests/modeladmin/tests.py\n+++ b/tests/modeladmin/tests.py\n@@ -153,6 +153,21 @@ class ModelAdminTests(TestCase):\n         self.assertIs(\n             ma.lookup_allowed(\"employee__department__code\", \"test_value\"), True\n         )\n+\n+    @isolate_apps(\"modeladmin\")\n+    def test_lookup_allowed_foreign_key_target_is_nonrelational(self):\n+        class CustomPrimaryKeyField(models.IntegerField):\n+            # Custom field attributes don't make a scalar field relational.\n+            path_infos = ()\n+\n+        class Target(models.Model):\n+            code = CustomPrimaryKeyField(primary_key=True)\n+\n+        class ReferencingModel(models.Model):\n+            target = models.ForeignKey(Target, models.CASCADE, to_field=\"code\")\n+\n+        ma = ModelAdmin(ReferencingModel, self.site)\n+        self.assertIs(ma.lookup_allowed(\"target__code\", \"1\"), True)\n \n     def test_field_arguments(self):\n         # If fields is specified, fieldsets_add and fieldsets_change should\n",
  "test_command": "cd /testbed && python tests/runtests.py modeladmin.tests.ModelAdminTests.test_lookup_allowed_foreign_key_target_is_nonrelational"
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
      "duration_seconds": 1.487,
      "log_path": "02_execution/attempt_01/candidate_a.log"
    },
    {
      "label": "candidate_b",
      "returncode": 0,
      "passed": true,
      "test_ran": true,
      "duration_seconds": 1.472,
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
Testing against Django installed in '/testbed/django' with up to 24 processes
Found 1 test(s).
System check identified no issues (0 silenced).
F
======================================================================
FAIL: test_lookup_allowed_foreign_key_target_is_nonrelational (modeladmin.tests.ModelAdminTests.test_lookup_allowed_foreign_key_target_is_nonrelational)
----------------------------------------------------------------------
Traceback (most recent call last):
  File "/testbed/django/test/utils.py", line 443, in inner
    return func(*args, **kwargs)
           ^^^^^^^^^^^^^^^^^^^^^
  File "/testbed/tests/modeladmin/tests.py", line 170, in test_lookup_allowed_foreign_key_target_is_nonrelational
    self.assertIs(ma.lookup_allowed("target__code", "1"), True)
AssertionError: False is not True

----------------------------------------------------------------------
Ran 1 test in 0.002s

FAILED (failures=1)
Destroying test database for alias 'default'...
[pipeline] test_exit_code=1

</candidate_a_execution_log>
<candidate_b_execution_log>
[pipeline] checking candidate patch
[pipeline] checking generated test patch
[pipeline] executing generated test command
Creating test database for alias 'default'...
Testing against Django installed in '/testbed/django' with up to 24 processes
Found 1 test(s).
System check identified no issues (0 silenced).
.
----------------------------------------------------------------------
Ran 1 test in 0.001s

OK
Destroying test database for alias 'default'...
[pipeline] test_exit_code=0

</candidate_b_execution_log>
<official_gold_patch>
diff --git a/django/contrib/admin/options.py b/django/contrib/admin/options.py
--- a/django/contrib/admin/options.py
+++ b/django/contrib/admin/options.py
@@ -461,12 +461,14 @@ def lookup_allowed(self, lookup, value):
                 # Lookups on nonexistent fields are ok, since they're ignored
                 # later.
                 break
-            # It is allowed to filter on values that would be found from local
-            # model anyways. For example, if you filter on employee__department__id,
-            # then the id value would be found already from employee__department_id.
             if not prev_field or (
                 prev_field.is_relation
-                and field not in prev_field.path_infos[-1].target_fields
+                and field not in model._meta.parents.values()
+                and field is not model._meta.auto_field
+                and (
+                    model._meta.auto_field is None
+                    or part not in getattr(prev_field, "to_fields", [])
+                )
             ):
                 relation_parts.append(part)
             if not getattr(field, "path_infos", None):

</official_gold_patch>
<gold_execution_summary>
{
  "label": "gold",
  "returncode": 1,
  "passed": false,
  "test_ran": true,
  "duration_seconds": 1.612,
  "log_path": "gold_execution/gold.log"
}
</gold_execution_summary>
<gold_failure_contract>
{
  "kind": "behavioral_test_failure",
  "required_substrings": [
    "test_lookup_allowed_foreign_key_target_is_nonrelational"
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
  "reference_log_sha256": "e44c5b3f6058962e6952820cd7183cc29871b9c8543f43151f36f169ec5589ba"
}
</gold_failure_contract>
<gold_execution_log>
[pipeline] checking candidate patch
[pipeline] checking generated test patch
[pipeline] executing generated test command
Creating test database for alias 'default'...
Testing against Django installed in '/testbed/django' with up to 24 processes
Found 1 test(s).
System check identified no issues (0 silenced).
F
======================================================================
FAIL: test_lookup_allowed_foreign_key_target_is_nonrelational (modeladmin.tests.ModelAdminTests.test_lookup_allowed_foreign_key_target_is_nonrelational)
----------------------------------------------------------------------
Traceback (most recent call last):
  File "/testbed/django/test/utils.py", line 443, in inner
    return func(*args, **kwargs)
           ^^^^^^^^^^^^^^^^^^^^^
  File "/testbed/tests/modeladmin/tests.py", line 170, in test_lookup_allowed_foreign_key_target_is_nonrelational
    self.assertIs(ma.lookup_allowed("target__code", "1"), True)
AssertionError: False is not True

----------------------------------------------------------------------
Ran 1 test in 0.001s

FAILED (failures=1)
Destroying test database for alias 'default'...
[pipeline] test_exit_code=1

</gold_execution_log>
