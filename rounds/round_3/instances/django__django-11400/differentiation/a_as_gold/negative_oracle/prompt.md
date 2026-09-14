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
Ordering problem in admin.RelatedFieldListFilter and admin.RelatedOnlyFieldListFilter
Description
	
RelatedFieldListFilter doesn't fall back to the ordering defined in Model._meta.ordering. 
Ordering gets set to an empty tuple in ​https://github.com/django/django/blob/2.2.1/django/contrib/admin/filters.py#L196 and unless ordering is defined on the related model's ModelAdmin class it stays an empty tuple. IMHO it should fall back to the ordering defined in the related model's Meta.ordering field.
RelatedOnlyFieldListFilter doesn't order the related model at all, even if ordering is defined on the related model's ModelAdmin class.
That's because the call to field.get_choices ​https://github.com/django/django/blob/2.2.1/django/contrib/admin/filters.py#L422 omits the ordering kwarg entirely.

</issue_statement>
<candidate_a_patch>
diff --git a/django/contrib/admin/filters.py b/django/contrib/admin/filters.py
index d65e01d5e2..07261929f0 100644
--- a/django/contrib/admin/filters.py
+++ b/django/contrib/admin/filters.py
@@ -419,4 +419,8 @@ FieldListFilter.register(lambda f: True, AllValuesFieldListFilter)
 class RelatedOnlyFieldListFilter(RelatedFieldListFilter):
     def field_choices(self, field, request, model_admin):
         pk_qs = model_admin.get_queryset(request).distinct().values_list('%s__pk' % self.field_path, flat=True)
-        return field.get_choices(include_blank=False, limit_choices_to={'pk__in': pk_qs})
+        ordering = ()
+        related_admin = model_admin.admin_site._registry.get(field.remote_field.model)
+        if related_admin is not None:
+            ordering = related_admin.get_ordering(request)
+        return field.get_choices(include_blank=False, limit_choices_to={'pk__in': pk_qs}, ordering=ordering)
diff --git a/django/db/models/fields/__init__.py b/django/db/models/fields/__init__.py
index 194514c900..8e84c6d2c2 100644
--- a/django/db/models/fields/__init__.py
+++ b/django/db/models/fields/__init__.py
@@ -825,6 +825,12 @@ class Field(RegisterLookupMixin):
             if hasattr(self.remote_field, 'get_related_field')
             else 'pk'
         )
+        if ordering:
+            ordering = tuple(ordering)
+        else:
+            # Fall back to the related model's ordering
+            related_model = self.remote_field.model
+            ordering = related_model._meta.ordering or ()
         return (blank_choice if include_blank else []) + [
             (choice_func(x), str(x))
             for x in rel_model._default_manager.complex_filter(limit_choices_to).order_by(*ordering)
diff --git a/django/db/models/fields/reverse_related.py b/django/db/models/fields/reverse_related.py
index eb6b934259..a007d989e5 100644
--- a/django/db/models/fields/reverse_related.py
+++ b/django/db/models/fields/reverse_related.py
@@ -122,6 +122,11 @@ class ForeignObjectRel(FieldCacheMixin):
         Analog of django.db.models.fields.Field.get_choices(), provided
         initially for utilization by RelatedFieldListFilter.
         """
+        if ordering:
+            ordering = tuple(ordering)
+        else:
+            # Fall back to the related model's ordering
+            ordering = self.related_model._meta.ordering or ()
         return (blank_choice if include_blank else []) + [
             (x.pk, str(x)) for x in self.related_model._default_manager.order_by(*ordering)
         ]

</candidate_a_patch>
<candidate_b_patch>
diff --git a/django/contrib/admin/filters.py b/django/contrib/admin/filters.py
index d65e01d5e2..07261929f0 100644
--- a/django/contrib/admin/filters.py
+++ b/django/contrib/admin/filters.py
@@ -419,4 +419,8 @@ FieldListFilter.register(lambda f: True, AllValuesFieldListFilter)
 class RelatedOnlyFieldListFilter(RelatedFieldListFilter):
     def field_choices(self, field, request, model_admin):
         pk_qs = model_admin.get_queryset(request).distinct().values_list('%s__pk' % self.field_path, flat=True)
-        return field.get_choices(include_blank=False, limit_choices_to={'pk__in': pk_qs})
+        ordering = ()
+        related_admin = model_admin.admin_site._registry.get(field.remote_field.model)
+        if related_admin is not None:
+            ordering = related_admin.get_ordering(request)
+        return field.get_choices(include_blank=False, limit_choices_to={'pk__in': pk_qs}, ordering=ordering)
diff --git a/django/db/models/fields/__init__.py b/django/db/models/fields/__init__.py
index 194514c900..f1ca8fafb0 100644
--- a/django/db/models/fields/__init__.py
+++ b/django/db/models/fields/__init__.py
@@ -825,6 +825,8 @@ class Field(RegisterLookupMixin):
             if hasattr(self.remote_field, 'get_related_field')
             else 'pk'
         )
+        if not ordering:
+            ordering = rel_model._meta.ordering or ()
         return (blank_choice if include_blank else []) + [
             (choice_func(x), str(x))
             for x in rel_model._default_manager.complex_filter(limit_choices_to).order_by(*ordering)
diff --git a/django/db/models/fields/reverse_related.py b/django/db/models/fields/reverse_related.py
index eb6b934259..030f0bf6e9 100644
--- a/django/db/models/fields/reverse_related.py
+++ b/django/db/models/fields/reverse_related.py
@@ -122,6 +122,8 @@ class ForeignObjectRel(FieldCacheMixin):
         Analog of django.db.models.fields.Field.get_choices(), provided
         initially for utilization by RelatedFieldListFilter.
         """
+        if not ordering:
+            ordering = self.related_model._meta.ordering or ()
         return (blank_choice if include_blank else []) + [
             (x.pk, str(x)) for x in self.related_model._default_manager.order_by(*ordering)
         ]

</candidate_b_patch>
<generated_test_proposal>
{
  "test_name": "test_get_choices_ordering_is_stable_during_filtering",
  "specification_gap": "A truthy explicit ordering iterable passed to Field.get_choices() is the ordering for that call and must remain stable while limit_choices_to is processed.",
  "input_description": "Create related Foo objects with a='a' and a='b', pass ordering=['a'], and pass a mapping-compatible limit_choices_to whose key enumeration changes the original list to ['-a'].",
  "expected_output": "get_choices() returns the two choices in ascending a order: foo1 ('a') followed by foo2 ('b'). The later mutation of the caller-owned list must not reverse this call's ordering.",
  "expected_passing_candidate": "candidate_a",
  "why_interesting": "This exercises two public get_choices() inputs together and exposes the only semantic difference between the patches: candidate_a snapshots a truthy ordering iterable before queryset filtering, while candidate_b expands the mutable iterable afterward and therefore returns reversed choices.",
  "test_patch": "diff --git a/tests/model_fields/tests.py b/tests/model_fields/tests.py\n--- a/tests/model_fields/tests.py\n+++ b/tests/model_fields/tests.py\n@@ -240,7 +240,30 @@ class GetChoicesOrderingTests(TestCase):\n             self.field.get_choices(include_blank=False, ordering=('-a',)),\n             [self.foo2, self.foo1]\n         )\n+\n+    def test_get_choices_ordering_is_stable_during_filtering(self):\n+        ordering = ['a']\n+\n+        class MutatingLimitChoicesTo:\n+            def __bool__(self):\n+                return True\n+\n+            def keys(self):\n+                ordering[:] = ['-a']\n+                return ()\n+\n+            def __getitem__(self, key):\n+                raise KeyError(key)\n+\n+        self.assertChoicesEqual(\n+            self.field.get_choices(\n+                include_blank=False,\n+                limit_choices_to=MutatingLimitChoicesTo(),\n+                ordering=ordering,\n+            ),\n+            [self.foo1, self.foo2],\n+        )\n \n     def test_get_choices_reverse_related_field(self):\n         self.assertChoicesEqual(\n             self.field.remote_field.get_choices(include_blank=False, ordering=('a',)),\n",
  "test_command": "cd /testbed && ./tests/runtests.py model_fields.tests.GetChoicesOrderingTests.test_get_choices_ordering_is_stable_during_filtering"
}
</generated_test_proposal>
<candidate_execution_summary>
{
  "attempt": 3,
  "separates": true,
  "required_passing_candidate": "candidate_a",
  "meets_acceptance_rule": true,
  "results": [
    {
      "label": "candidate_a",
      "returncode": 0,
      "passed": true,
      "test_ran": true,
      "duration_seconds": 1.73,
      "log_path": "02_execution/attempt_03/candidate_a.log"
    },
    {
      "label": "candidate_b",
      "returncode": 1,
      "passed": false,
      "test_ran": true,
      "duration_seconds": 1.558,
      "log_path": "02_execution/attempt_03/candidate_b.log"
    }
  ]
}
</candidate_execution_summary>
<candidate_a_execution_log>
[pipeline] checking candidate patch
[pipeline] checking generated test patch
[pipeline] executing generated test command
Creating test database for alias 'default'...
.Testing against Django installed in '/testbed/django' with up to 24 processes
System check identified no issues (0 silenced).

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
F
======================================================================
FAIL: test_get_choices_ordering_is_stable_during_filtering (model_fields.tests.GetChoicesOrderingTests)
----------------------------------------------------------------------
Traceback (most recent call last):
  File "/testbed/tests/model_fields/tests.py", line 264, in test_get_choices_ordering_is_stable_during_filtering
    [self.foo1, self.foo2],
  File "/testbed/tests/model_fields/tests.py", line 232, in assertChoicesEqual
    self.assertEqual(choices, [(obj.pk, str(obj)) for obj in objs])
AssertionError: Lists differ: [(2, 'Foo object (2)'), (1, 'Foo object (1)')] != [(1, 'Foo object (1)'), (2, 'Foo object (2)')]

First differing element 0:
(2, 'Foo object (2)')
(1, 'Foo object (1)')

- [(2, 'Foo object (2)'), (1, 'Foo object (1)')]
?   ^               ^      ^               ^

+ [(1, 'Foo object (1)'), (2, 'Foo object (2)')]
?   ^               ^      ^               ^


----------------------------------------------------------------------
Ran 1 test in 0.002s

FAILED (failures=1)
Destroying test database for alias 'default'...
Testing against Django installed in '/testbed/django' with up to 24 processes
System check identified no issues (0 silenced).
[pipeline] test_exit_code=1

</candidate_b_execution_log>
<official_gold_patch>
diff --git a/django/contrib/admin/filters.py b/django/contrib/admin/filters.py
--- a/django/contrib/admin/filters.py
+++ b/django/contrib/admin/filters.py
@@ -193,11 +193,17 @@ def has_output(self):
     def expected_parameters(self):
         return [self.lookup_kwarg, self.lookup_kwarg_isnull]
 
-    def field_choices(self, field, request, model_admin):
-        ordering = ()
+    def field_admin_ordering(self, field, request, model_admin):
+        """
+        Return the model admin's ordering for related field, if provided.
+        """
         related_admin = model_admin.admin_site._registry.get(field.remote_field.model)
         if related_admin is not None:
-            ordering = related_admin.get_ordering(request)
+            return related_admin.get_ordering(request)
+        return ()
+
+    def field_choices(self, field, request, model_admin):
+        ordering = self.field_admin_ordering(field, request, model_admin)
         return field.get_choices(include_blank=False, ordering=ordering)
 
     def choices(self, changelist):
@@ -419,4 +425,5 @@ def choices(self, changelist):
 class RelatedOnlyFieldListFilter(RelatedFieldListFilter):
     def field_choices(self, field, request, model_admin):
         pk_qs = model_admin.get_queryset(request).distinct().values_list('%s__pk' % self.field_path, flat=True)
-        return field.get_choices(include_blank=False, limit_choices_to={'pk__in': pk_qs})
+        ordering = self.field_admin_ordering(field, request, model_admin)
+        return field.get_choices(include_blank=False, limit_choices_to={'pk__in': pk_qs}, ordering=ordering)
diff --git a/django/db/models/fields/__init__.py b/django/db/models/fields/__init__.py
--- a/django/db/models/fields/__init__.py
+++ b/django/db/models/fields/__init__.py
@@ -825,9 +825,11 @@ def get_choices(self, include_blank=True, blank_choice=BLANK_CHOICE_DASH, limit_
             if hasattr(self.remote_field, 'get_related_field')
             else 'pk'
         )
+        qs = rel_model._default_manager.complex_filter(limit_choices_to)
+        if ordering:
+            qs = qs.order_by(*ordering)
         return (blank_choice if include_blank else []) + [
-            (choice_func(x), str(x))
-            for x in rel_model._default_manager.complex_filter(limit_choices_to).order_by(*ordering)
+            (choice_func(x), str(x)) for x in qs
         ]
 
     def value_to_string(self, obj):
diff --git a/django/db/models/fields/reverse_related.py b/django/db/models/fields/reverse_related.py
--- a/django/db/models/fields/reverse_related.py
+++ b/django/db/models/fields/reverse_related.py
@@ -122,8 +122,11 @@ def get_choices(self, include_blank=True, blank_choice=BLANK_CHOICE_DASH, orderi
         Analog of django.db.models.fields.Field.get_choices(), provided
         initially for utilization by RelatedFieldListFilter.
         """
+        qs = self.related_model._default_manager.all()
+        if ordering:
+            qs = qs.order_by(*ordering)
         return (blank_choice if include_blank else []) + [
-            (x.pk, str(x)) for x in self.related_model._default_manager.order_by(*ordering)
+            (x.pk, str(x)) for x in qs
         ]
 
     def is_hidden(self):

</official_gold_patch>
<gold_execution_summary>
{
  "label": "gold",
  "returncode": 1,
  "passed": false,
  "test_ran": true,
  "duration_seconds": 1.565,
  "log_path": "gold_execution/gold.log"
}
</gold_execution_summary>
<gold_failure_contract>
{
  "kind": "behavioral_test_failure",
  "required_substrings": [
    "test_get_choices_ordering_is_stable_during_filtering"
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
  "reference_log_sha256": "827cf6eb5856b6ca74f9ed55bd4d5ec547173c1b95c4798c05cd6b4039b0e178"
}
</gold_failure_contract>
<gold_execution_log>
[pipeline] checking candidate patch
[pipeline] checking generated test patch
[pipeline] executing generated test command
Creating test database for alias 'default'...
FTesting against Django installed in '/testbed/django' with up to 24 processes
System check identified no issues (0 silenced).

======================================================================
FAIL: test_get_choices_ordering_is_stable_during_filtering (model_fields.tests.GetChoicesOrderingTests)
----------------------------------------------------------------------
Traceback (most recent call last):
  File "/testbed/tests/model_fields/tests.py", line 264, in test_get_choices_ordering_is_stable_during_filtering
    [self.foo1, self.foo2],
  File "/testbed/tests/model_fields/tests.py", line 232, in assertChoicesEqual
    self.assertEqual(choices, [(obj.pk, str(obj)) for obj in objs])
AssertionError: Lists differ: [(2, 'Foo object (2)'), (1, 'Foo object (1)')] != [(1, 'Foo object (1)'), (2, 'Foo object (2)')]

First differing element 0:
(2, 'Foo object (2)')
(1, 'Foo object (1)')

- [(2, 'Foo object (2)'), (1, 'Foo object (1)')]
?   ^               ^      ^               ^

+ [(1, 'Foo object (1)'), (2, 'Foo object (2)')]
?   ^               ^      ^               ^


----------------------------------------------------------------------
Ran 1 test in 0.002s

FAILED (failures=1)
Destroying test database for alias 'default'...
[pipeline] test_exit_code=1

</gold_execution_log>
