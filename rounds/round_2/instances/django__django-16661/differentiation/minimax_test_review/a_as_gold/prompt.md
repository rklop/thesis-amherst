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
diff --git a/django/contrib/admin/options.py b/django/contrib/admin/options.py
index 5240b07cd6..352739e869 100644
--- a/django/contrib/admin/options.py
+++ b/django/contrib/admin/options.py
@@ -466,19 +466,48 @@ class BaseModelAdmin(metaclass=forms.MediaDefiningClass):
             # then the id value would be found already from employee__department_id.
             if not prev_field or (
                 prev_field.is_relation
-                and field not in prev_field.path_infos[-1].target_fields
+                and (
+                    field not in prev_field.path_infos[-1].target_fields
+                    or (
+                        # Add the field to relation_parts if it's a traversable
+                        # relation, even if it's a target field of the previous
+                        # relation (e.g., a OneToOneField used as primary key).
+                        getattr(field, "path_infos", None)
+                        and field.path_infos
+                    )
+                )
             ):
                 relation_parts.append(part)
             if not getattr(field, "path_infos", None):
                 # This is not a relational field, so further parts
                 # must be transforms.
                 break
+            if not field.path_infos:
+                # This field is a relation but not traversable (e.g., a
+                # OneToOneField used as primary key with empty path_info).
+                # Don't allow further traversal.
+                break
             prev_field = field
             model = field.path_infos[-1].to_opts.model
 
-        if len(relation_parts) <= 1:
-            # Either a local field filter, or no fields at all.
+        if len(relation_parts) > 1:
+            # More than one relation part, need to check if it's valid.
+            pass
+        elif prev_field is None:
+            # No fields at all.
             return True
+        else:
+            # Only one relation part, check if it was a multi-part lookup
+            # that ended early (e.g., due to non-traversable relation).
+            # If so, we need to validate it properly.
+            lookup_parts = lookup.split(LOOKUP_SEP)
+            if len(lookup_parts) > 1:
+                # This is a multi-part lookup that ended early.
+                # Go through the validation logic below.
+                pass
+            else:
+                # Single local field filter.
+                return True
         valid_lookups = {self.date_hierarchy}
         for filter_item in self.list_filter:
             if isinstance(filter_item, type) and issubclass(

</candidate_a_patch>
<candidate_b_patch>
diff --git a/django/contrib/admin/options.py b/django/contrib/admin/options.py
index 5240b07cd6..f0905b3a86 100644
--- a/django/contrib/admin/options.py
+++ b/django/contrib/admin/options.py
@@ -466,8 +466,21 @@ class BaseModelAdmin(metaclass=forms.MediaDefiningClass):
             # then the id value would be found already from employee__department_id.
             if not prev_field or (
                 prev_field.is_relation
-                and field not in prev_field.path_infos[-1].target_fields
+                and (
+                    field not in prev_field.path_infos[-1].target_fields
+                    or hasattr(field, "path_infos")
+                )
+            ):
+                relation_parts.append(part)
+            elif (
+                prev_field.is_relation
+                and field in prev_field.path_infos[-1].target_fields
+                and hasattr(field, "path_infos")
+                and not field.path_infos
             ):
+                # The field is in target_fields of the previous relation but
+                # has no path_info (not traversable). Include it in relation_parts
+                # so the lookup is checked against list_filter.
                 relation_parts.append(part)
             if not getattr(field, "path_infos", None):
                 # This is not a relational field, so further parts

</candidate_b_patch>
<generated_test_proposal>
{
  "test_name": "test_lookup_allowed_nontraversable_primary_key_relation",
  "specification_gap": "When a listed relation is followed by its target primary-key field, that terminal key lookup remains authorized even if the key is itself a relation with no further traversal path. It must not be treated as a separate relation requiring its own list_filter entry.",
  "input_description": "Define Restaurant.place as a non-traversable OneToOneField primary key, reference Restaurant from Waiter.restaurant, configure list_filter = [\"restaurant\"], and call lookup_allowed(\"restaurant__place\", \"test_value\").",
  "expected_output": "ModelAdmin.lookup_allowed() returns True because restaurant__place addresses the target key already represented by the authorized restaurant relation.",
  "expected_passing_candidate": "candidate_a",
  "why_interesting": "This exercises the empty-path boundary of the issue's foreign-key-target collapsing rule. Candidate_a retains restaurant as the effective lookup, while candidate_b counts place as another relation and incorrectly returns False.",
  "test_patch": "diff --git a/tests/modeladmin/tests.py b/tests/modeladmin/tests.py\n--- a/tests/modeladmin/tests.py\n+++ b/tests/modeladmin/tests.py\n@@ -153,6 +153,31 @@ class ModelAdminTests(TestCase):\n         self.assertIs(\n             ma.lookup_allowed(\"employee__department__code\", \"test_value\"), True\n         )\n \n+    @isolate_apps(\"modeladmin\")\n+    def test_lookup_allowed_nontraversable_primary_key_relation(self):\n+        class NonTraversableOneToOneField(models.OneToOneField):\n+            def get_path_info(self, filtered_relation=None):\n+                return []\n+\n+        class Place(models.Model):\n+            pass\n+\n+        class Restaurant(models.Model):\n+            place = NonTraversableOneToOneField(\n+                Place, models.CASCADE, primary_key=True\n+            )\n+\n+        class Waiter(models.Model):\n+            restaurant = models.ForeignKey(Restaurant, models.CASCADE)\n+\n+        class WaiterAdmin(ModelAdmin):\n+            list_filter = [\"restaurant\"]\n+\n+        ma = WaiterAdmin(Waiter, self.site)\n+        self.assertIs(\n+            ma.lookup_allowed(\"restaurant__place\", \"test_value\"), True\n+        )\n+\n     def test_field_arguments(self):\n         # If fields is specified, fieldsets_add and fieldsets_change should\n",
  "test_command": "python tests/runtests.py modeladmin.tests.ModelAdminTests.test_lookup_allowed_nontraversable_primary_key_relation"
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
      "duration_seconds": 1.565,
      "log_path": "02_execution/attempt_01/candidate_a.log"
    },
    {
      "label": "candidate_b",
      "returncode": 1,
      "passed": false,
      "test_ran": true,
      "duration_seconds": 1.519,
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
.
----------------------------------------------------------------------
Ran 1 test in 0.001s

OK
Destroying test database for alias 'default'...
[pipeline] test_exit_code=0

</candidate_a_execution_log>
<candidate_b_execution_log>
[pipeline] checking candidate patch
[pipeline] checking generated test patch
[pipeline] executing generated test command
Creating test database for alias 'default'...
Testing against Django installed in '/testbed/django' with up to 24 processes
Found 1 test(s).
System check identified no issues (0 silenced).
F
======================================================================
FAIL: test_lookup_allowed_nontraversable_primary_key_relation (modeladmin.tests.ModelAdminTests.test_lookup_allowed_nontraversable_primary_key_relation)
----------------------------------------------------------------------
Traceback (most recent call last):
  File "/testbed/django/test/utils.py", line 443, in inner
    return func(*args, **kwargs)
           ^^^^^^^^^^^^^^^^^^^^^
  File "/testbed/tests/modeladmin/tests.py", line 178, in test_lookup_allowed_nontraversable_primary_key_relation
    self.assertIs(
AssertionError: False is not True

----------------------------------------------------------------------
Ran 1 test in 0.001s

FAILED (failures=1)
Destroying test database for alias 'default'...
[pipeline] test_exit_code=1

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
  "duration_seconds": 1.505,
  "log_path": "gold_execution/gold.log"
}
</gold_execution_summary>
<gold_failure_contract>
{
  "kind": "behavioral_test_failure",
  "required_substrings": [
    "test_lookup_allowed_nontraversable_primary_key_relation"
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
  "reference_log_sha256": "b77e698ccd5b2ab13982d244bfa1956eb2aa58e87585ca13618704da23a93db6"
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
FAIL: test_lookup_allowed_nontraversable_primary_key_relation (modeladmin.tests.ModelAdminTests.test_lookup_allowed_nontraversable_primary_key_relation)
----------------------------------------------------------------------
Traceback (most recent call last):
  File "/testbed/django/test/utils.py", line 443, in inner
    return func(*args, **kwargs)
           ^^^^^^^^^^^^^^^^^^^^^
  File "/testbed/tests/modeladmin/tests.py", line 178, in test_lookup_allowed_nontraversable_primary_key_relation
    self.assertIs(
AssertionError: False is not True

----------------------------------------------------------------------
Ran 1 test in 0.001s

FAILED (failures=1)
Destroying test database for alias 'default'...
[pipeline] test_exit_code=1

</gold_execution_log>
