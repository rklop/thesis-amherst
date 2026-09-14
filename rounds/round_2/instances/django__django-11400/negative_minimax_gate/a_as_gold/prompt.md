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
diff --git a/django/contrib/admin/filters.py b/django/contrib/admin/filters.py
index d65e01d5e2..937b8f2f07 100644
--- a/django/contrib/admin/filters.py
+++ b/django/contrib/admin/filters.py
@@ -194,10 +194,10 @@ class RelatedFieldListFilter(FieldListFilter):
         return [self.lookup_kwarg, self.lookup_kwarg_isnull]
 
     def field_choices(self, field, request, model_admin):
-        ordering = ()
+        ordering = field.remote_field.model._meta.ordering or ()
         related_admin = model_admin.admin_site._registry.get(field.remote_field.model)
         if related_admin is not None:
-            ordering = related_admin.get_ordering(request)
+            ordering = related_admin.get_ordering(request) or ordering
         return field.get_choices(include_blank=False, ordering=ordering)
 
     def choices(self, changelist):
@@ -419,4 +419,8 @@ FieldListFilter.register(lambda f: True, AllValuesFieldListFilter)
 class RelatedOnlyFieldListFilter(RelatedFieldListFilter):
     def field_choices(self, field, request, model_admin):
         pk_qs = model_admin.get_queryset(request).distinct().values_list('%s__pk' % self.field_path, flat=True)
-        return field.get_choices(include_blank=False, limit_choices_to={'pk__in': pk_qs})
+        ordering = field.remote_field.model._meta.ordering or ()
+        related_admin = model_admin.admin_site._registry.get(field.remote_field.model)
+        if related_admin is not None:
+            ordering = related_admin.get_ordering(request) or ordering
+        return field.get_choices(include_blank=False, limit_choices_to={'pk__in': pk_qs}, ordering=ordering)
diff --git a/django/db/models/fields/__init__.py b/django/db/models/fields/__init__.py
index 194514c900..b2ad53798b 100644
--- a/django/db/models/fields/__init__.py
+++ b/django/db/models/fields/__init__.py
@@ -825,6 +825,7 @@ class Field(RegisterLookupMixin):
             if hasattr(self.remote_field, 'get_related_field')
             else 'pk'
         )
+        ordering = ordering or rel_model._meta.ordering
         return (blank_choice if include_blank else []) + [
             (choice_func(x), str(x))
             for x in rel_model._default_manager.complex_filter(limit_choices_to).order_by(*ordering)
diff --git a/django/db/models/fields/reverse_related.py b/django/db/models/fields/reverse_related.py
index eb6b934259..33fd024df1 100644
--- a/django/db/models/fields/reverse_related.py
+++ b/django/db/models/fields/reverse_related.py
@@ -122,6 +122,7 @@ class ForeignObjectRel(FieldCacheMixin):
         Analog of django.db.models.fields.Field.get_choices(), provided
         initially for utilization by RelatedFieldListFilter.
         """
+        ordering = ordering or self.related_model._meta.ordering
         return (blank_choice if include_blank else []) + [
             (x.pk, str(x)) for x in self.related_model._default_manager.order_by(*ordering)
         ]

</candidate_a_patch>
<candidate_b_patch>
diff --git a/django/contrib/admin/filters.py b/django/contrib/admin/filters.py
index d65e01d5e2..f6710f7e68 100644
--- a/django/contrib/admin/filters.py
+++ b/django/contrib/admin/filters.py
@@ -198,6 +198,8 @@ class RelatedFieldListFilter(FieldListFilter):
         related_admin = model_admin.admin_site._registry.get(field.remote_field.model)
         if related_admin is not None:
             ordering = related_admin.get_ordering(request)
+        if not ordering:
+            ordering = field.remote_field.model._meta.ordering or ()
         return field.get_choices(include_blank=False, ordering=ordering)
 
     def choices(self, changelist):
@@ -419,4 +421,10 @@ FieldListFilter.register(lambda f: True, AllValuesFieldListFilter)
 class RelatedOnlyFieldListFilter(RelatedFieldListFilter):
     def field_choices(self, field, request, model_admin):
         pk_qs = model_admin.get_queryset(request).distinct().values_list('%s__pk' % self.field_path, flat=True)
-        return field.get_choices(include_blank=False, limit_choices_to={'pk__in': pk_qs})
+        ordering = ()
+        related_admin = model_admin.admin_site._registry.get(field.remote_field.model)
+        if related_admin is not None:
+            ordering = related_admin.get_ordering(request)
+        if not ordering:
+            ordering = field.remote_field.model._meta.ordering or ()
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
  "test_name": "test_get_choices_falls_back_to_falsy_nonempty_model_ordering",
  "specification_gap": "A related model's Meta.ordering is an ordering sequence, not merely a boolean flag. A valid tuple subclass may be false-valued while still containing ordering terms; get_choices() must preserve and apply those terms when falling back to Meta.ordering.",
  "input_description": "Create Foo rows with values 'a' and then 'b'. Configure Foo.Meta.ordering with a false-valued tuple containing '-a', then call Bar's foreign-key field get_choices(include_blank=False) without explicit ordering.",
  "expected_output": "The choices are ordered by '-a': Foo 'b' appears before Foo 'a'.",
  "expected_passing_candidate": "candidate_a",
  "why_interesting": "candidate_a passes the configured ordering iterable to order_by(). candidate_b boolean-normalizes it to an empty tuple, clearing the ordering and returning the rows in insertion order in the standard SQLite test environment.",
  "test_patch": "diff --git a/tests/model_fields/models.py b/tests/model_fields/models.py\n--- a/tests/model_fields/models.py\n+++ b/tests/model_fields/models.py\n@@ -17,12 +17,20 @@ try:\n     from PIL import Image\n except ImportError:\n     Image = None\n \n \n+class FalsyOrdering(tuple):\n+    def __bool__(self):\n+        return False\n+\n+\n class Foo(models.Model):\n     a = models.CharField(max_length=10)\n     d = models.DecimalField(max_digits=5, decimal_places=3)\n \n+    class Meta:\n+        ordering = FalsyOrdering(('-a',))\n+\n \n def get_foo():\n     return Foo.objects.get(id=1).pk\n diff --git a/tests/model_fields/tests.py b/tests/model_fields/tests.py\n--- a/tests/model_fields/tests.py\n+++ b/tests/model_fields/tests.py\n@@ -240,6 +240,12 @@ class GetChoicesOrderingTests(TestCase):\n             self.field.get_choices(include_blank=False, ordering=('-a',)),\n             [self.foo2, self.foo1]\n         )\n \n+    def test_get_choices_falls_back_to_falsy_nonempty_model_ordering(self):\n+        self.assertChoicesEqual(\n+            self.field.get_choices(include_blank=False),\n+            [self.foo2, self.foo1],\n+        )\n+\n     def test_get_choices_reverse_related_field(self):\n         self.assertChoicesEqual(\n             self.field.remote_field.get_choices(include_blank=False, ordering=('a',)),\n",
  "test_command": "cd /testbed && python tests/runtests.py model_fields.tests.GetChoicesOrderingTests.test_get_choices_falls_back_to_falsy_nonempty_model_ordering"
}
</generated_test_proposal>
<candidate_execution_summary>
{
  "attempt": 2,
  "separates": true,
  "required_passing_candidate": "candidate_a",
  "meets_acceptance_rule": true,
  "results": [
    {
      "label": "candidate_a",
      "returncode": 0,
      "passed": true,
      "test_ran": true,
      "duration_seconds": 1.645,
      "log_path": "02_execution/attempt_02/candidate_a.log"
    },
    {
      "label": "candidate_b",
      "returncode": 1,
      "passed": false,
      "test_ran": true,
      "duration_seconds": 1.733,
      "log_path": "02_execution/attempt_02/candidate_b.log"
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
FAIL: test_get_choices_falls_back_to_falsy_nonempty_model_ordering (model_fields.tests.GetChoicesOrderingTests)
----------------------------------------------------------------------
Traceback (most recent call last):
  File "/testbed/tests/model_fields/tests.py", line 247, in test_get_choices_falls_back_to_falsy_nonempty_model_ordering
    [self.foo2, self.foo1],
  File "/testbed/tests/model_fields/tests.py", line 232, in assertChoicesEqual
    self.assertEqual(choices, [(obj.pk, str(obj)) for obj in objs])
AssertionError: Lists differ: [(1, 'Foo object (1)'), (2, 'Foo object (2)')] != [(2, 'Foo object (2)'), (1, 'Foo object (1)')]

First differing element 0:
(1, 'Foo object (1)')
(2, 'Foo object (2)')

- [(1, 'Foo object (1)'), (2, 'Foo object (2)')]
?   ^               ^      ^               ^

+ [(2, 'Foo object (2)'), (1, 'Foo object (1)')]
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
  "duration_seconds": 1.788,
  "log_path": "gold_execution/gold.log"
}
</gold_execution_summary>
<gold_failure_contract>
{
  "kind": "behavioral_test_failure",
  "required_substrings": [
    "test_get_choices_falls_back_to_falsy_nonempty_model_ordering"
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
  "reference_log_sha256": "2916d13e83d7faf14cc14af0ce33008b66c9f9ed7b07d8d47755bb62c5686541"
}
</gold_failure_contract>
<gold_execution_log>
[pipeline] checking candidate patch
[pipeline] checking generated test patch
[pipeline] executing generated test command
Creating test database for alias 'default'...
F
======================================================================
FAIL: test_get_choices_falls_back_to_falsy_nonempty_model_ordering (model_fields.tests.GetChoicesOrderingTests)
----------------------------------------------------------------------
Traceback (most recent call last):
  File "/testbed/tests/model_fields/tests.py", line 247, in test_get_choices_falls_back_to_falsy_nonempty_model_ordering
    [self.foo2, self.foo1],
  File "/testbed/tests/model_fields/tests.py", line 232, in assertChoicesEqual
    self.assertEqual(choices, [(obj.pk, str(obj)) for obj in objs])
AssertionError: Lists differ: [(1, 'Foo object (1)'), (2, 'Foo object (2)')] != [(2, 'Foo object (2)'), (1, 'Foo object (1)')]

First differing element 0:
(1, 'Foo object (1)')
(2, 'Foo object (2)')

- [(1, 'Foo object (1)'), (2, 'Foo object (2)')]
?   ^               ^      ^               ^

+ [(2, 'Foo object (2)'), (1, 'Foo object (1)')]
?   ^               ^      ^               ^


----------------------------------------------------------------------
Ran 1 test in 0.002s

FAILED (failures=1)
Destroying test database for alias 'default'...
Testing against Django installed in '/testbed/django' with up to 24 processes
System check identified no issues (0 silenced).
[pipeline] test_exit_code=1

</gold_execution_log>
