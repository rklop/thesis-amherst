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

<original_test_patch>
diff --git a/tests/modeladmin/tests.py b/tests/modeladmin/tests.py
--- a/tests/modeladmin/tests.py
+++ b/tests/modeladmin/tests.py
@@ -154,6 +154,35 @@ class EmployeeProfileAdmin(ModelAdmin):
             ma.lookup_allowed("employee__department__code", "test_value"), True
         )
 
+    @isolate_apps("modeladmin")
+    def test_lookup_allowed_foreign_primary(self):
+        class Country(models.Model):
+            name = models.CharField(max_length=256)
+
+        class Place(models.Model):
+            country = models.ForeignKey(Country, models.CASCADE)
+
+        class Restaurant(models.Model):
+            place = models.OneToOneField(Place, models.CASCADE, primary_key=True)
+
+        class Waiter(models.Model):
+            restaurant = models.ForeignKey(Restaurant, models.CASCADE)
+
+        class WaiterAdmin(ModelAdmin):
+            list_filter = [
+                "restaurant__place__country",
+                "restaurant__place__country__name",
+            ]
+
+        ma = WaiterAdmin(Waiter, self.site)
+        self.assertIs(ma.lookup_allowed("restaurant__place__country", "1"), True)
+        self.assertIs(
+            ma.lookup_allowed("restaurant__place__country__id__exact", "1"), True
+        )
+        self.assertIs(
+            ma.lookup_allowed("restaurant__place__country__name", "test_value"), True
+        )
+
     def test_field_arguments(self):
         # If fields is specified, fieldsets_add and fieldsets_change should
         # just stick the fields into a formsets structure and return it.

</original_test_patch>

<candidate_a_patch>
diff --git a/django/contrib/admin/options.py b/django/contrib/admin/options.py
index 5240b07cd6..cc53ff1515 100644
--- a/django/contrib/admin/options.py
+++ b/django/contrib/admin/options.py
@@ -465,8 +465,12 @@ class BaseModelAdmin(metaclass=forms.MediaDefiningClass):
             # model anyways. For example, if you filter on employee__department__id,
             # then the id value would be found already from employee__department_id.
             if not prev_field or (
-                prev_field.is_relation
-                and field not in prev_field.path_infos[-1].target_fields
+                field.is_relation
+                or (
+                    not field.is_relation
+                    and prev_field.is_relation
+                    and field not in prev_field.path_infos[-1].target_fields
+                )
             ):
                 relation_parts.append(part)
             if not getattr(field, "path_infos", None):

</candidate_a_patch>

<candidate_b_patch>
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

</candidate_b_patch>

<generated_test_proposal>
{
  "test_name": "test_lookup_allowed_concrete_parent_link",
  "specification_gap": "ModelAdmin.lookup_allowed() must treat an explicit concrete-inheritance parent-link traversal as equivalent to direct access to the inherited field. Candidate A incorrectly treats every relational segment as significant, while candidate B preserves this canonicalization.",
  "input_description": "A Referrer model points to a multi-table-inheritance Child. Its ModelAdmin allows the inherited-field filter \"child__label\". Call lookup_allowed() with the equivalent explicit path \"child__parent_ptr__label\".",
  "expected_output": "lookup_allowed() returns True because Child.label and Child.parent_ptr.label denote the same inherited field path.",
  "expected_passing_candidate": "candidate_b",
  "why_interesting": "This covers a regression boundary adjacent to the reported relational-primary-key bug: ordinary relation primary keys must be distinguished from concrete-inheritance parent links. The diff was validated with git apply --check against the pristine repository.",
  "test_patch": "diff --git a/tests/modeladmin/tests.py b/tests/modeladmin/tests.py\n--- a/tests/modeladmin/tests.py\n+++ b/tests/modeladmin/tests.py\n@@ -152,6 +152,26 @@ class ModelAdminTests(TestCase):\n         # OneToOneField and ForeignKey\n         self.assertIs(\n             ma.lookup_allowed(\"employee__department__code\", \"test_value\"), True\n         )\n \n+    @isolate_apps(\"modeladmin\")\n+    def test_lookup_allowed_concrete_parent_link(self):\n+        class Parent(models.Model):\n+            label = models.CharField(max_length=20)\n+\n+        class Child(Parent):\n+            pass\n+\n+        class Referrer(models.Model):\n+            child = models.ForeignKey(Child, models.CASCADE)\n+\n+        class ReferrerAdmin(ModelAdmin):\n+            list_filter = [\"child__label\"]\n+\n+        ma = ReferrerAdmin(Referrer, self.site)\n+        self.assertIs(\n+            ma.lookup_allowed(\"child__parent_ptr__label\", \"test_value\"),\n+            True,\n+        )\n+\n     def test_field_arguments(self):\n         # If fields is specified, fieldsets_add and fieldsets_change should\n         # just stick the fields into a formsets structure and return it.\n",
  "test_command": "python tests/runtests.py modeladmin.tests.ModelAdminTests.test_lookup_allowed_concrete_parent_link"
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
Creating test database for alias 'default'...
Testing against Django installed in '/testbed/django' with up to 24 processes
Found 1 test(s).
System check identified no issues (0 silenced).
F
======================================================================
FAIL: test_lookup_allowed_concrete_parent_link (modeladmin.tests.ModelAdminTests.test_lookup_allowed_concrete_parent_link)
----------------------------------------------------------------------
Traceback (most recent call last):
  File "/testbed/django/test/utils.py", line 443, in inner
    return func(*args, **kwargs)
           ^^^^^^^^^^^^^^^^^^^^^
  File "/testbed/tests/modeladmin/tests.py", line 172, in test_lookup_allowed_concrete_parent_link
    self.assertIs(
AssertionError: False is not True

----------------------------------------------------------------------
Ran 1 test in 0.001s

FAILED (failures=1)
Destroying test database for alias 'default'...
[pipeline] test_exit_code=1

```

## candidate_b
returncode=0 passed=True test_ran=True
log=02_execution/attempt_02/candidate_b.log
```
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

```
</validated_execution>
