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
Ordering problem in admin.RelatedFieldListFilter and admin.RelatedOnlyFieldListFilter
Description
	
RelatedFieldListFilter doesn't fall back to the ordering defined in Model._meta.ordering. 
Ordering gets set to an empty tuple in ​https://github.com/django/django/blob/2.2.1/django/contrib/admin/filters.py#L196 and unless ordering is defined on the related model's ModelAdmin class it stays an empty tuple. IMHO it should fall back to the ordering defined in the related model's Meta.ordering field.
RelatedOnlyFieldListFilter doesn't order the related model at all, even if ordering is defined on the related model's ModelAdmin class.
That's because the call to field.get_choices ​https://github.com/django/django/blob/2.2.1/django/contrib/admin/filters.py#L422 omits the ordering kwarg entirely.

</issue_statement>

<original_test_patch>
diff --git a/tests/admin_filters/tests.py b/tests/admin_filters/tests.py
--- a/tests/admin_filters/tests.py
+++ b/tests/admin_filters/tests.py
@@ -591,6 +591,22 @@ class BookAdmin(ModelAdmin):
         expected = [(self.john.pk, 'John Blue'), (self.jack.pk, 'Jack Red')]
         self.assertEqual(filterspec.lookup_choices, expected)
 
+    def test_relatedfieldlistfilter_foreignkey_default_ordering(self):
+        """RelatedFieldListFilter ordering respects Model.ordering."""
+        class BookAdmin(ModelAdmin):
+            list_filter = ('employee',)
+
+        self.addCleanup(setattr, Employee._meta, 'ordering', Employee._meta.ordering)
+        Employee._meta.ordering = ('name',)
+        modeladmin = BookAdmin(Book, site)
+
+        request = self.request_factory.get('/')
+        request.user = self.alfred
+        changelist = modeladmin.get_changelist_instance(request)
+        filterspec = changelist.get_filters(request)[0][0]
+        expected = [(self.jack.pk, 'Jack Red'), (self.john.pk, 'John Blue')]
+        self.assertEqual(filterspec.lookup_choices, expected)
+
     def test_relatedfieldlistfilter_manytomany(self):
         modeladmin = BookAdmin(Book, site)
 
@@ -696,6 +712,23 @@ def test_relatedfieldlistfilter_reverse_relationships(self):
         filterspec = changelist.get_filters(request)[0]
         self.assertEqual(len(filterspec), 0)
 
+    def test_relatedfieldlistfilter_reverse_relationships_default_ordering(self):
+        self.addCleanup(setattr, Book._meta, 'ordering', Book._meta.ordering)
+        Book._meta.ordering = ('title',)
+        modeladmin = CustomUserAdmin(User, site)
+
+        request = self.request_factory.get('/')
+        request.user = self.alfred
+        changelist = modeladmin.get_changelist_instance(request)
+        filterspec = changelist.get_filters(request)[0][0]
+        expected = [
+            (self.bio_book.pk, 'Django: a biography'),
+            (self.djangonaut_book.pk, 'Djangonaut: an art of living'),
+            (self.guitar_book.pk, 'Guitar for dummies'),
+            (self.django_book.pk, 'The Django Book')
+        ]
+        self.assertEqual(filterspec.lookup_choices, expected)
+
     def test_relatedonlyfieldlistfilter_foreignkey(self):
         modeladmin = BookAdminRelatedOnlyFilter(Book, site)
 
@@ -708,6 +741,57 @@ def test_relatedonlyfieldlistfilter_foreignkey(self):
         expected = [(self.alfred.pk, 'alfred'), (self.bob.pk, 'bob')]
         self.assertEqual(sorted(filterspec.lookup_choices), sorted(expected))
 
+    def test_relatedonlyfieldlistfilter_foreignkey_ordering(self):
+        """RelatedOnlyFieldListFilter ordering respects ModelAdmin.ordering."""
+        class EmployeeAdminWithOrdering(ModelAdmin):
+            ordering = ('name',)
+
+        class BookAdmin(ModelAdmin):
+            list_filter = (
+                ('employee', RelatedOnlyFieldListFilter),
+            )
+
+        albert = Employee.objects.create(name='Albert Green', department=self.dev)
+        self.djangonaut_book.employee = albert
+        self.djangonaut_book.save()
+        self.bio_book.employee = self.jack
+        self.bio_book.save()
+
+        site.register(Employee, EmployeeAdminWithOrdering)
+        self.addCleanup(lambda: site.unregister(Employee))
+        modeladmin = BookAdmin(Book, site)
+
+        request = self.request_factory.get('/')
+        request.user = self.alfred
+        changelist = modeladmin.get_changelist_instance(request)
+        filterspec = changelist.get_filters(request)[0][0]
+        expected = [(albert.pk, 'Albert Green'), (self.jack.pk, 'Jack Red')]
+        self.assertEqual(filterspec.lookup_choices, expected)
+
+    def test_relatedonlyfieldlistfilter_foreignkey_default_ordering(self):
+        """RelatedOnlyFieldListFilter ordering respects Meta.ordering."""
+        class BookAdmin(ModelAdmin):
+            list_filter = (
+                ('employee', RelatedOnlyFieldListFilter),
+            )
+
+        albert = Employee.objects.create(name='Albert Green', department=self.dev)
+        self.djangonaut_book.employee = albert
+        self.djangonaut_book.save()
+        self.bio_book.employee = self.jack
+        self.bio_book.save()
+
+        self.addCleanup(setattr, Employee._meta, 'ordering', Employee._meta.ordering)
+        Employee._meta.ordering = ('name',)
+        modeladmin = BookAdmin(Book, site)
+
+        request = self.request_factory.get('/')
+        request.user = self.alfred
+        changelist = modeladmin.get_changelist_instance(request)
+        filterspec = changelist.get_filters(request)[0][0]
+        expected = [(albert.pk, 'Albert Green'), (self.jack.pk, 'Jack Red')]
+        self.assertEqual(filterspec.lookup_choices, expected)
+
     def test_relatedonlyfieldlistfilter_underscorelookup_foreignkey(self):
         Department.objects.create(code='TEST', description='Testing')
         self.djangonaut_book.employee = self.john
diff --git a/tests/model_fields/tests.py b/tests/model_fields/tests.py
--- a/tests/model_fields/tests.py
+++ b/tests/model_fields/tests.py
@@ -222,9 +222,9 @@ class GetChoicesOrderingTests(TestCase):
 
     @classmethod
     def setUpTestData(cls):
-        cls.foo1 = Foo.objects.create(a='a', d='12.34')
+        cls.foo1 = Foo.objects.create(a='a', d='12.35')
         cls.foo2 = Foo.objects.create(a='b', d='12.34')
-        cls.bar1 = Bar.objects.create(a=cls.foo1, b='a')
+        cls.bar1 = Bar.objects.create(a=cls.foo1, b='b')
         cls.bar2 = Bar.objects.create(a=cls.foo2, b='a')
         cls.field = Bar._meta.get_field('a')
 
@@ -241,6 +241,14 @@ def test_get_choices(self):
             [self.foo2, self.foo1]
         )
 
+    def test_get_choices_default_ordering(self):
+        self.addCleanup(setattr, Foo._meta, 'ordering', Foo._meta.ordering)
+        Foo._meta.ordering = ('d',)
+        self.assertChoicesEqual(
+            self.field.get_choices(include_blank=False),
+            [self.foo2, self.foo1]
+        )
+
     def test_get_choices_reverse_related_field(self):
         self.assertChoicesEqual(
             self.field.remote_field.get_choices(include_blank=False, ordering=('a',)),
@@ -250,3 +258,11 @@ def test_get_choices_reverse_related_field(self):
             self.field.remote_field.get_choices(include_blank=False, ordering=('-a',)),
             [self.bar2, self.bar1]
         )
+
+    def test_get_choices_reverse_related_field_default_ordering(self):
+        self.addCleanup(setattr, Bar._meta, 'ordering', Bar._meta.ordering)
+        Bar._meta.ordering = ('b',)
+        self.assertChoicesEqual(
+            self.field.remote_field.get_choices(include_blank=False),
+            [self.bar2, self.bar1]
+        )

</original_test_patch>

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

</candidate_a_patch>

<candidate_b_patch>
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

</candidate_b_patch>

<generated_test_proposal>
{
  "test_name": "test_get_choices_uses_default_manager_ordering",
  "specification_gap": "When no explicit ordering is passed to Field.get_choices(), it should preserve the effective ordering of the related model's default-manager queryset. This includes a custom default manager that intentionally overrides Meta.ordering. Candidate B preserves that queryset ordering, while candidate A replaces it with _meta.ordering.",
  "input_description": "Foo declares ascending Meta.ordering by \"a\", but its default manager explicitly orders by descending \"a\". Create Foo rows \"a\" then \"b\" and call Bar's ForeignKey.get_choices(include_blank=False) without an ordering argument.",
  "expected_output": "The choices are returned in effective default-manager order: foo2 (\"b\") followed by foo1 (\"a\"). Candidate A instead returns ascending Meta order.",
  "expected_passing_candidate": "candidate_b",
  "why_interesting": "Opposite manager and Meta orderings provide a deterministic boundary case that tests the public no-explicit-override behavior without relying on incidental database row order or admin implementation details.",
  "test_patch": "diff --git a/tests/model_fields/models.py b/tests/model_fields/models.py\n--- a/tests/model_fields/models.py\n+++ b/tests/model_fields/models.py\n@@ -20,9 +20,18 @@ except ImportError:\n     Image = None\n \n \n+class FooManager(models.Manager):\n+    def get_queryset(self):\n+        return super().get_queryset().order_by('-a')\n+\n+\n class Foo(models.Model):\n     a = models.CharField(max_length=10)\n     d = models.DecimalField(max_digits=5, decimal_places=3)\n+    objects = FooManager()\n+\n+    class Meta:\n+        ordering = ('a',)\n \n \n def get_foo():\ndiff --git a/tests/model_fields/tests.py b/tests/model_fields/tests.py\n--- a/tests/model_fields/tests.py\n+++ b/tests/model_fields/tests.py\n@@ -231,6 +231,12 @@ class GetChoicesOrderingTests(TestCase):\n     def assertChoicesEqual(self, choices, objs):\n         self.assertEqual(choices, [(obj.pk, str(obj)) for obj in objs])\n \n+    def test_get_choices_uses_default_manager_ordering(self):\n+        self.assertChoicesEqual(\n+            self.field.get_choices(include_blank=False),\n+            [self.foo2, self.foo1],\n+        )\n+\n     def test_get_choices(self):\n         self.assertChoicesEqual(\n             self.field.get_choices(include_blank=False, ordering=('a',)),\n",
  "test_command": "cd /testbed && ./tests/runtests.py model_fields.tests.GetChoicesOrderingTests.test_get_choices_uses_default_manager_ordering"
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
FTesting against Django installed in '/testbed/django' with up to 24 processes
System check identified no issues (0 silenced).

======================================================================
FAIL: test_get_choices_uses_default_manager_ordering (model_fields.tests.GetChoicesOrderingTests)
----------------------------------------------------------------------
Traceback (most recent call last):
  File "/testbed/tests/model_fields/tests.py", line 237, in test_get_choices_uses_default_manager_ordering
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
.Testing against Django installed in '/testbed/django' with up to 24 processes
System check identified no issues (0 silenced).

----------------------------------------------------------------------
Ran 1 test in 0.001s

OK
Destroying test database for alias 'default'...
[pipeline] test_exit_code=0

```
</validated_execution>
