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
limit_choices_to on a ForeignKey can render duplicate options in formfield
Description
	
If you pass a Q object as limit_choices_to on a ForeignKey field involving a join, you may end up with duplicate options in your form.
See regressiontest in patch for a clear view on the problem.

</issue_statement>

<original_test_patch>
diff --git a/tests/model_forms/models.py b/tests/model_forms/models.py
--- a/tests/model_forms/models.py
+++ b/tests/model_forms/models.py
@@ -411,9 +411,14 @@ class StumpJoke(models.Model):
         Character,
         models.CASCADE,
         limit_choices_to=today_callable_dict,
-        related_name="+",
+        related_name='jokes',
     )
-    has_fooled_today = models.ManyToManyField(Character, limit_choices_to=today_callable_q, related_name="+")
+    has_fooled_today = models.ManyToManyField(
+        Character,
+        limit_choices_to=today_callable_q,
+        related_name='jokes_today',
+    )
+    funny = models.BooleanField(default=False)
 
 
 # Model for #13776
diff --git a/tests/model_forms/tests.py b/tests/model_forms/tests.py
--- a/tests/model_forms/tests.py
+++ b/tests/model_forms/tests.py
@@ -16,6 +16,7 @@
 )
 from django.template import Context, Template
 from django.test import SimpleTestCase, TestCase, skipUnlessDBFeature
+from django.test.utils import isolate_apps
 
 from .models import (
     Article, ArticleStatus, Author, Author1, Award, BetterWriter, BigInt, Book,
@@ -2829,6 +2830,72 @@ def test_callable_called_each_time_form_is_instantiated(self):
             StumpJokeForm()
             self.assertEqual(today_callable_dict.call_count, 3)
 
+    @isolate_apps('model_forms')
+    def test_limit_choices_to_no_duplicates(self):
+        joke1 = StumpJoke.objects.create(
+            funny=True,
+            most_recently_fooled=self.threepwood,
+        )
+        joke2 = StumpJoke.objects.create(
+            funny=True,
+            most_recently_fooled=self.threepwood,
+        )
+        joke3 = StumpJoke.objects.create(
+            funny=True,
+            most_recently_fooled=self.marley,
+        )
+        StumpJoke.objects.create(funny=False, most_recently_fooled=self.marley)
+        joke1.has_fooled_today.add(self.marley, self.threepwood)
+        joke2.has_fooled_today.add(self.marley)
+        joke3.has_fooled_today.add(self.marley, self.threepwood)
+
+        class CharacterDetails(models.Model):
+            character1 = models.ForeignKey(
+                Character,
+                models.CASCADE,
+                limit_choices_to=models.Q(
+                    jokes__funny=True,
+                    jokes_today__funny=True,
+                ),
+                related_name='details_fk_1',
+            )
+            character2 = models.ForeignKey(
+                Character,
+                models.CASCADE,
+                limit_choices_to={
+                    'jokes__funny': True,
+                    'jokes_today__funny': True,
+                },
+                related_name='details_fk_2',
+            )
+            character3 = models.ManyToManyField(
+                Character,
+                limit_choices_to=models.Q(
+                    jokes__funny=True,
+                    jokes_today__funny=True,
+                ),
+                related_name='details_m2m_1',
+            )
+
+        class CharacterDetailsForm(forms.ModelForm):
+            class Meta:
+                model = CharacterDetails
+                fields = '__all__'
+
+        form = CharacterDetailsForm()
+        self.assertCountEqual(
+            form.fields['character1'].queryset,
+            [self.marley, self.threepwood],
+        )
+        self.assertCountEqual(
+            form.fields['character2'].queryset,
+            [self.marley, self.threepwood],
+        )
+        self.assertCountEqual(
+            form.fields['character3'].queryset,
+            [self.marley, self.threepwood],
+        )
+
 
 class FormFieldCallbackTests(SimpleTestCase):
 

</original_test_patch>

<candidate_a_patch>
diff --git a/django/db/models/fields/__init__.py b/django/db/models/fields/__init__.py
index 2bce85beef..aaaf4f2002 100644
--- a/django/db/models/fields/__init__.py
+++ b/django/db/models/fields/__init__.py
@@ -880,7 +880,7 @@ class Field(RegisterLookupMixin):
             if hasattr(self.remote_field, 'get_related_field')
             else 'pk'
         )
-        qs = rel_model._default_manager.complex_filter(limit_choices_to)
+        qs = rel_model._default_manager.complex_filter(limit_choices_to).distinct()
         if ordering:
             qs = qs.order_by(*ordering)
         return (blank_choice if include_blank else []) + [
diff --git a/django/db/models/fields/reverse_related.py b/django/db/models/fields/reverse_related.py
index 12160add7f..29bfc8e8e3 100644
--- a/django/db/models/fields/reverse_related.py
+++ b/django/db/models/fields/reverse_related.py
@@ -150,7 +150,7 @@ class ForeignObjectRel(FieldCacheMixin):
         initially for utilization by RelatedFieldListFilter.
         """
         limit_choices_to = limit_choices_to or self.limit_choices_to
-        qs = self.related_model._default_manager.complex_filter(limit_choices_to)
+        qs = self.related_model._default_manager.complex_filter(limit_choices_to).distinct()
         if ordering:
             qs = qs.order_by(*ordering)
         return (blank_choice if include_blank else []) + [
diff --git a/django/forms/models.py b/django/forms/models.py
index 5d115458a1..f993bb9c98 100644
--- a/django/forms/models.py
+++ b/django/forms/models.py
@@ -100,7 +100,7 @@ def apply_limit_choices_to_to_formfield(formfield):
     if hasattr(formfield, 'queryset') and hasattr(formfield, 'get_limit_choices_to'):
         limit_choices_to = formfield.get_limit_choices_to()
         if limit_choices_to is not None:
-            formfield.queryset = formfield.queryset.complex_filter(limit_choices_to)
+            formfield.queryset = formfield.queryset.complex_filter(limit_choices_to).distinct()
 
 
 def fields_for_model(model, fields=None, exclude=None, widgets=None,

</candidate_a_patch>

<candidate_b_patch>
diff --git a/django/forms/models.py b/django/forms/models.py
--- a/django/forms/models.py
+++ b/django/forms/models.py
@@ -97,10 +97,18 @@ def model_to_dict(instance, fields=None, exclude=None):
 
 def apply_limit_choices_to_to_formfield(formfield):
     """Apply limit_choices_to to the formfield's queryset if needed."""
+    from django.db.models import Exists, OuterRef, Q
     if hasattr(formfield, 'queryset') and hasattr(formfield, 'get_limit_choices_to'):
         limit_choices_to = formfield.get_limit_choices_to()
-        if limit_choices_to is not None:
-            formfield.queryset = formfield.queryset.complex_filter(limit_choices_to)
+        if limit_choices_to:
+            complex_filter = limit_choices_to
+            if not isinstance(complex_filter, Q):
+                complex_filter = Q(**limit_choices_to)
+            complex_filter &= Q(pk=OuterRef('pk'))
+            # Use Exists() to avoid potential duplicates.
+            formfield.queryset = formfield.queryset.filter(
+                Exists(formfield.queryset.model._base_manager.filter(complex_filter)),
+            )
 
 
 def fields_for_model(model, fields=None, exclude=None, widgets=None,

</candidate_b_patch>

<generated_test_proposal>
{
  "test_name": "test_get_choices_removes_duplicates",
  "specification_gap": "Duplicate suppression should also hold when choices are generated through the public relation Field.get_choices() API, not only when a ModelForm constructs its queryset.",
  "input_description": "Create a second Bar pointing to foo1, then call the ForeignKey field's get_choices(include_blank=False) with Q(bars__isnull=False). The join matches foo1 twice and foo2 once.",
  "expected_output": "The returned choices are exactly [(foo1.pk, str(foo1)), (foo2.pk, str(foo2))], with one tuple per related object.",
  "expected_passing_candidate": "candidate_a",
  "why_interesting": "Candidate A applies distinct() to Field.get_choices(), while candidate B only changes ModelForm limit_choices_to handling. This exercises the same user-visible no-duplicate invariant through an alternate public entry point and uses the Q-object join variant named in the issue.",
  "test_patch": "diff --git a/tests/model_fields/tests.py b/tests/model_fields/tests.py\n--- a/tests/model_fields/tests.py\n+++ b/tests/model_fields/tests.py\n@@ -341,6 +341,14 @@ class GetChoicesLimitChoicesToTests(TestCase):\n         self.assertChoicesEqual(\n             self.field.get_choices(include_blank=False, limit_choices_to={}),\n             [self.foo1, self.foo2],\n         )\n \n+    def test_get_choices_removes_duplicates(self):\n+        Bar.objects.create(a=self.foo1, b='another')\n+        choices = self.field.get_choices(\n+            include_blank=False, limit_choices_to=models.Q(bars__isnull=False),\n+            ordering=('pk',),\n+        )\n+        self.assertChoicesEqual(choices, [self.foo1, self.foo2])\n+\n     def test_get_choices_reverse_related_field(self):\n         field = self.field.remote_field\n         self.assertChoicesEqual(\n",
  "test_command": "./tests/runtests.py model_fields.tests.GetChoicesLimitChoicesToTests.test_get_choices_removes_duplicates"
}
</generated_test_proposal>

<validated_execution>
## candidate_a
returncode=0 passed=True test_ran=True
log=02_execution/attempt_02/candidate_a.log
```
[pipeline] checking candidate patch
[pipeline] checking generated test patch
[pipeline] executing generated test command
Testing against Django installed in '/testbed/django' with up to 24 processes
Creating test database for alias 'default'...
System check identified no issues (3 silenced).
.
----------------------------------------------------------------------
Ran 1 test in 0.002s

OK
Destroying test database for alias 'default'...
[pipeline] test_exit_code=0

```

## candidate_b
returncode=1 passed=False test_ran=True
log=02_execution/attempt_02/candidate_b.log
```
[pipeline] checking candidate patch
[pipeline] checking generated test patch
[pipeline] executing generated test command
Creating test database for alias 'default'...
Testing against Django installed in '/testbed/django' with up to 24 processes
System check identified no issues (3 silenced).
F
======================================================================
FAIL: test_get_choices_removes_duplicates (model_fields.tests.GetChoicesLimitChoicesToTests)
----------------------------------------------------------------------
Traceback (most recent call last):
  File "/testbed/tests/model_fields/tests.py", line 352, in test_get_choices_removes_duplicates
    self.assertChoicesEqual(choices, [self.foo1, self.foo2])
  File "/testbed/tests/model_fields/tests.py", line 334, in assertChoicesEqual
    self.assertEqual(choices, [(obj.pk, str(obj)) for obj in objs])
AssertionError: Lists differ: [(1, 'Foo object (1)'), (1, 'Foo object (1)'), (2, 'Foo object (2)')] != [(1, 'Foo object (1)'), (2, 'Foo object (2)')]

First differing element 1:
(1, 'Foo object (1)')
(2, 'Foo object (2)')

First list contains 1 additional elements.
First extra element 2:
(2, 'Foo object (2)')

- [(1, 'Foo object (1)'), (1, 'Foo object (1)'), (2, 'Foo object (2)')]
?                          ^              -----------------------

+ [(1, 'Foo object (1)'), (2, 'Foo object (2)')]
?                          ^


----------------------------------------------------------------------
Ran 1 test in 0.002s

FAILED (failures=1)
Destroying test database for alias 'default'...
[pipeline] test_exit_code=1

```
</validated_execution>
