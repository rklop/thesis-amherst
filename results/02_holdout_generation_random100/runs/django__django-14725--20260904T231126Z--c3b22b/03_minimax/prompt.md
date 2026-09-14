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
Provide a way for model formsets to disallow new object creation
Description
	
Model formsets don't provide a way to create an "edit only" view of objects. We see users trying to use extra=0 to accomplish this, but that's not reliable as extra is merely meant for the extra number of forms to display. You can add more forms with Javascript (or just send additional post data).

</issue_statement>

<original_test_patch>
diff --git a/tests/model_formsets/tests.py b/tests/model_formsets/tests.py
--- a/tests/model_formsets/tests.py
+++ b/tests/model_formsets/tests.py
@@ -1771,6 +1771,73 @@ def test_initial_form_count_empty_data(self):
         formset = AuthorFormSet({})
         self.assertEqual(formset.initial_form_count(), 0)
 
+    def test_edit_only(self):
+        charles = Author.objects.create(name='Charles Baudelaire')
+        AuthorFormSet = modelformset_factory(Author, fields='__all__', edit_only=True)
+        data = {
+            'form-TOTAL_FORMS': '2',
+            'form-INITIAL_FORMS': '0',
+            'form-MAX_NUM_FORMS': '0',
+            'form-0-name': 'Arthur Rimbaud',
+            'form-1-name': 'Walt Whitman',
+        }
+        formset = AuthorFormSet(data)
+        self.assertIs(formset.is_valid(), True)
+        formset.save()
+        self.assertSequenceEqual(Author.objects.all(), [charles])
+        data = {
+            'form-TOTAL_FORMS': '2',
+            'form-INITIAL_FORMS': '1',
+            'form-MAX_NUM_FORMS': '0',
+            'form-0-id': charles.pk,
+            'form-0-name': 'Arthur Rimbaud',
+            'form-1-name': 'Walt Whitman',
+        }
+        formset = AuthorFormSet(data)
+        self.assertIs(formset.is_valid(), True)
+        formset.save()
+        charles.refresh_from_db()
+        self.assertEqual(charles.name, 'Arthur Rimbaud')
+        self.assertSequenceEqual(Author.objects.all(), [charles])
+
+    def test_edit_only_inlineformset_factory(self):
+        charles = Author.objects.create(name='Charles Baudelaire')
+        book = Book.objects.create(author=charles, title='Les Paradis Artificiels')
+        AuthorFormSet = inlineformset_factory(
+            Author, Book, can_delete=False, fields='__all__', edit_only=True,
+        )
+        data = {
+            'book_set-TOTAL_FORMS': '4',
+            'book_set-INITIAL_FORMS': '1',
+            'book_set-MAX_NUM_FORMS': '0',
+            'book_set-0-id': book.pk,
+            'book_set-0-title': 'Les Fleurs du Mal',
+            'book_set-0-author': charles.pk,
+            'book_set-1-title': 'Flowers of Evil',
+            'book_set-1-author': charles.pk,
+        }
+        formset = AuthorFormSet(data, instance=charles)
+        self.assertIs(formset.is_valid(), True)
+        formset.save()
+        book.refresh_from_db()
+        self.assertEqual(book.title, 'Les Fleurs du Mal')
+        self.assertSequenceEqual(Book.objects.all(), [book])
+
+    def test_edit_only_object_outside_of_queryset(self):
+        charles = Author.objects.create(name='Charles Baudelaire')
+        walt = Author.objects.create(name='Walt Whitman')
+        data = {
+            'form-TOTAL_FORMS': '1',
+            'form-INITIAL_FORMS': '1',
+            'form-0-id': walt.pk,
+            'form-0-name': 'Parth Patil',
+        }
+        AuthorFormSet = modelformset_factory(Author, fields='__all__', edit_only=True)
+        formset = AuthorFormSet(data, queryset=Author.objects.filter(pk=charles.pk))
+        self.assertIs(formset.is_valid(), True)
+        formset.save()
+        self.assertCountEqual(Author.objects.all(), [charles, walt])
+
 
 class TestModelFormsetOverridesTroughFormMeta(TestCase):
     def test_modelformset_factory_widgets(self):

</original_test_patch>

<candidate_a_patch>
diff --git a/django/forms/models.py b/django/forms/models.py
index 7effb202e3..31144ba6c5 100644
--- a/django/forms/models.py
+++ b/django/forms/models.py
@@ -568,6 +568,7 @@ class BaseModelFormSet(BaseFormSet):
     A ``FormSet`` for editing a queryset and/or adding new objects to it.
     """
     model = None
+    edit_only = False
 
     # Set of fields that must be unique among forms of this set.
     unique_fields = set()
@@ -813,6 +814,8 @@ class BaseModelFormSet(BaseFormSet):
 
     def save_new_objects(self, commit=True):
         self.new_objects = []
+        if self.edit_only:
+            return self.new_objects
         for form in self.extra_forms:
             if not form.has_changed():
                 continue
@@ -875,7 +878,8 @@ def modelformset_factory(model, form=ModelForm, formfield_callback=None,
                          widgets=None, validate_max=False, localized_fields=None,
                          labels=None, help_texts=None, error_messages=None,
                          min_num=None, validate_min=False, field_classes=None,
-                         absolute_max=None, can_delete_extra=True, renderer=None):
+                         absolute_max=None, can_delete_extra=True, renderer=None,
+                         edit_only=False):
     """Return a FormSet class for the given Django model class."""
     meta = getattr(form, 'Meta', None)
     if (getattr(meta, 'fields', fields) is None and
@@ -896,6 +900,7 @@ def modelformset_factory(model, form=ModelForm, formfield_callback=None,
                               absolute_max=absolute_max, can_delete_extra=can_delete_extra,
                               renderer=renderer)
     FormSet.model = model
+    FormSet.edit_only = edit_only
     return FormSet
 
 
@@ -1076,7 +1081,8 @@ def inlineformset_factory(parent_model, model, form=ModelForm,
                           widgets=None, validate_max=False, localized_fields=None,
                           labels=None, help_texts=None, error_messages=None,
                           min_num=None, validate_min=False, field_classes=None,
-                          absolute_max=None, can_delete_extra=True, renderer=None):
+                          absolute_max=None, can_delete_extra=True, renderer=None,
+                          edit_only=False):
     """
     Return an ``InlineFormSet`` for the given kwargs.
 
@@ -1109,6 +1115,7 @@ def inlineformset_factory(parent_model, model, form=ModelForm,
         'absolute_max': absolute_max,
         'can_delete_extra': can_delete_extra,
         'renderer': renderer,
+        'edit_only': edit_only,
     }
     FormSet = modelformset_factory(model, **kwargs)
     FormSet.fk = fk

</candidate_a_patch>

<candidate_b_patch>
diff --git a/django/forms/models.py b/django/forms/models.py
--- a/django/forms/models.py
+++ b/django/forms/models.py
@@ -676,7 +676,10 @@ def save_m2m():
                 for form in self.saved_forms:
                     form.save_m2m()
             self.save_m2m = save_m2m
-        return self.save_existing_objects(commit) + self.save_new_objects(commit)
+        if self.edit_only:
+            return self.save_existing_objects(commit)
+        else:
+            return self.save_existing_objects(commit) + self.save_new_objects(commit)
 
     save.alters_data = True
 
@@ -875,7 +878,8 @@ def modelformset_factory(model, form=ModelForm, formfield_callback=None,
                          widgets=None, validate_max=False, localized_fields=None,
                          labels=None, help_texts=None, error_messages=None,
                          min_num=None, validate_min=False, field_classes=None,
-                         absolute_max=None, can_delete_extra=True, renderer=None):
+                         absolute_max=None, can_delete_extra=True, renderer=None,
+                         edit_only=False):
     """Return a FormSet class for the given Django model class."""
     meta = getattr(form, 'Meta', None)
     if (getattr(meta, 'fields', fields) is None and
@@ -896,6 +900,7 @@ def modelformset_factory(model, form=ModelForm, formfield_callback=None,
                               absolute_max=absolute_max, can_delete_extra=can_delete_extra,
                               renderer=renderer)
     FormSet.model = model
+    FormSet.edit_only = edit_only
     return FormSet
 
 
@@ -1076,7 +1081,8 @@ def inlineformset_factory(parent_model, model, form=ModelForm,
                           widgets=None, validate_max=False, localized_fields=None,
                           labels=None, help_texts=None, error_messages=None,
                           min_num=None, validate_min=False, field_classes=None,
-                          absolute_max=None, can_delete_extra=True, renderer=None):
+                          absolute_max=None, can_delete_extra=True, renderer=None,
+                          edit_only=False):
     """
     Return an ``InlineFormSet`` for the given kwargs.
 
@@ -1109,6 +1115,7 @@ def inlineformset_factory(parent_model, model, form=ModelForm,
         'absolute_max': absolute_max,
         'can_delete_extra': can_delete_extra,
         'renderer': renderer,
+        'edit_only': edit_only,
     }
     FormSet = modelformset_factory(model, **kwargs)
     FormSet.fk = fk

</candidate_b_patch>

<generated_test_proposal>
{
  "test_name": "test_edit_only_skips_custom_save_new_objects",
  "specification_gap": "The edit_only guarantee must remain authoritative for custom model formset classes supplied through the public formset= extension point. Candidate_a guards only BaseModelFormSet.save_new_objects(), so an existing override can still create objects. Candidate_b skips the new-object saving phase entirely.",
  "input_description": "Submit one valid new Author form to a model formset configured with edit_only=True and a custom BaseModelFormSet subclass whose save_new_objects() implements conventional new-form saving.",
  "expected_output": "The formset is valid, save() returns an empty list, and no Author is inserted into the database.",
  "expected_passing_candidate": "candidate_b",
  "why_interesting": "This covers a public customization entry point and checks the externally observable edit-only invariant. Candidate_b bypasses the custom creation hook and passes; candidate_a dispatches to the override, creates an Author, and fails. The unified diff was validated with git apply --check against the pristine repository.",
  "test_patch": "diff --git a/tests/model_formsets/tests.py b/tests/model_formsets/tests.py\n--- a/tests/model_formsets/tests.py\n+++ b/tests/model_formsets/tests.py\n@@ -506,6 +506,33 @@ class ModelFormsetTest(TestCase):\n         AuthorFormSet = modelformset_factory(Author, fields='__all__', formset=BaseAuthorFormSet)\n         formset = AuthorFormSet()\n         self.assertEqual(len(formset.get_queryset()), 1)\n \n+    def test_edit_only_skips_custom_save_new_objects(self):\n+        class CustomAuthorFormSet(BaseModelFormSet):\n+            def save_new_objects(self, commit=True):\n+                self.new_objects = [\n+                    self.save_new(form, commit=commit)\n+                    for form in self.extra_forms\n+                    if form.has_changed()\n+                ]\n+                return self.new_objects\n+\n+        AuthorFormSet = modelformset_factory(\n+            Author,\n+            fields='__all__',\n+            formset=CustomAuthorFormSet,\n+            edit_only=True,\n+        )\n+        data = {\n+            'form-TOTAL_FORMS': '1',\n+            'form-INITIAL_FORMS': '0',\n+            'form-MAX_NUM_FORMS': '',\n+            'form-0-name': 'Ada Lovelace',\n+        }\n+        formset = AuthorFormSet(data)\n+        self.assertTrue(formset.is_valid())\n+        self.assertEqual(formset.save(), [])\n+        self.assertFalse(Author.objects.exists())\n+\n     def test_model_inheritance(self):\n         BetterAuthorFormSet = modelformset_factory(BetterAuthor, fields=\"__all__\")\n         formset = BetterAuthorFormSet()\n",
  "test_command": "./tests/runtests.py model_formsets.tests.ModelFormsetTest.test_edit_only_skips_custom_save_new_objects"
}
</generated_test_proposal>

<validated_execution>
## candidate_a
returncode=1 passed=False test_ran=True
log=02_execution/attempt_03/candidate_a.log
```
[pipeline] checking candidate patch
[pipeline] checking generated test patch
[pipeline] executing generated test command
Creating test database for alias 'default'...
Testing against Django installed in '/testbed/django' with up to 24 processes
Found 1 test(s).
System check identified no issues (1 silenced).
F
======================================================================
FAIL: test_edit_only_skips_custom_save_new_objects (model_formsets.tests.ModelFormsetTest)
----------------------------------------------------------------------
Traceback (most recent call last):
  File "/testbed/tests/model_formsets/tests.py", line 534, in test_edit_only_skips_custom_save_new_objects
    self.assertEqual(formset.save(), [])
AssertionError: Lists differ: [<Author: Ada Lovelace>] != []

First list contains 1 additional elements.
First extra element 0:
<Author: Ada Lovelace>

- [<Author: Ada Lovelace>]
+ []

----------------------------------------------------------------------
Ran 1 test in 0.002s

FAILED (failures=1)
Destroying test database for alias 'default'...
[pipeline] test_exit_code=1

```

## candidate_b
returncode=0 passed=True test_ran=True
log=02_execution/attempt_03/candidate_b.log
```
[pipeline] checking candidate patch
[pipeline] checking generated test patch
[pipeline] executing generated test command
Creating test database for alias 'default'...
Testing against Django installed in '/testbed/django' with up to 24 processes
Found 1 test(s).
System check identified no issues (1 silenced).
.
----------------------------------------------------------------------
Ran 1 test in 0.001s

OK
Destroying test database for alias 'default'...
[pipeline] test_exit_code=0

```
</validated_execution>
