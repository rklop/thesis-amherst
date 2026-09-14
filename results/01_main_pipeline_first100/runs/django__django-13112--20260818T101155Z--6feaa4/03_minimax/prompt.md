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
makemigrations crashes for ForeignKey with mixed-case app name.
Description
	
When i run "python3 manage.py migrate" on Django 3.1b1 shows me that error (Please, note that the code works well in 3.0)
ValueError: The field DJ_RegLogin.Content.category was declared with a lazy reference to 'dj_reglogin.category', but app 'dj_reglogin' isn't installed.
model.py (Conflict Part)
class Category(models.Model):
	title = models.CharField(max_length=100, db_index=True)
	slug = models.SlugField(max_length=100, db_index=True)
	class Meta:
		verbose_name = 'Category'
		verbose_name_plural = 'Categories'
	def __str__(self):
		return self.title
	def get_absolute_url(self):
		return reverse('view_blog_category', None, kwargs={'slug': self.slug})
class Content(models.Model):
	title = models.CharField(max_length=100, unique=True)
	slug = models.SlugField(max_length=100, unique=True)
	body = RichTextField(config_name='default')
	posted = models.DateTimeField(db_index=True, auto_now_add=True)
	sites = models.ManyToManyField(Site)
	ip = models.GenericIPAddressField(editable=False)
	category = models.ForeignKey(Category, on_delete=models.CASCADE)
	user = models.ForeignKey(User, on_delete=models.CASCADE, null=False, blank=False, editable=False)
	status = models.CharField(max_length=10, choices=STATUS_CHOICES, default='draft')
	def __str__(self):
		return self.title
	def get_absolute_url(self):
		return reverse('view_blog_post', None, kwargs={'slug': self.slug})
settings.py (Related to issue part)
INSTALLED_APPS = [
	'DJ_RegLogin',
	'django.contrib.admin',
	'django.contrib.auth',
	'django.contrib.contenttypes',
	'django.contrib.sessions',
	'django.contrib.messages',
	'django.contrib.staticfiles',
	'social_django',
	'ckeditor',
	'django.contrib.sites',
	'django.contrib.flatpages',
	'django.contrib.sitemaps',
]
apps.py
from django.apps import AppConfig
class DJ_RegLoginConfig(AppConfig):
	name = 'DJ_RegLogin'
	verbose_name = "Contents"

</issue_statement>

<original_test_patch>
diff --git a/tests/migrations/test_state.py b/tests/migrations/test_state.py
--- a/tests/migrations/test_state.py
+++ b/tests/migrations/test_state.py
@@ -867,6 +867,34 @@ class Meta:
         with self.assertRaisesMessage(ValueError, msg):
             project_state.apps
 
+    def test_reference_mixed_case_app_label(self):
+        new_apps = Apps()
+
+        class Author(models.Model):
+            class Meta:
+                app_label = 'MiXedCase_migrations'
+                apps = new_apps
+
+        class Book(models.Model):
+            author = models.ForeignKey(Author, models.CASCADE)
+
+            class Meta:
+                app_label = 'MiXedCase_migrations'
+                apps = new_apps
+
+        class Magazine(models.Model):
+            authors = models.ManyToManyField(Author)
+
+            class Meta:
+                app_label = 'MiXedCase_migrations'
+                apps = new_apps
+
+        project_state = ProjectState()
+        project_state.add_model(ModelState.from_model(Author))
+        project_state.add_model(ModelState.from_model(Book))
+        project_state.add_model(ModelState.from_model(Magazine))
+        self.assertEqual(len(project_state.apps.get_models()), 3)
+
     def test_real_apps(self):
         """
         Including real apps can resolve dangling FK errors.

</original_test_patch>

<candidate_a_patch>
diff --git a/django/db/models/fields/related.py b/django/db/models/fields/related.py
index c2aea9c730..3ad1e4606d 100644
--- a/django/db/models/fields/related.py
+++ b/django/db/models/fields/related.py
@@ -582,7 +582,11 @@ class ForeignObject(RelatedField):
         if self.remote_field.parent_link:
             kwargs['parent_link'] = self.remote_field.parent_link
         if isinstance(self.remote_field.model, str):
-            kwargs['to'] = self.remote_field.model.lower()
+            if '.' in self.remote_field.model:
+                app_label, model_name = self.remote_field.model.rsplit('.', 1)
+                kwargs['to'] = '%s.%s' % (app_label, model_name.lower())
+            else:
+                kwargs['to'] = self.remote_field.model.lower()
         else:
             kwargs['to'] = self.remote_field.model._meta.label_lower
         # If swappable is True, then see if we're actually pointing to the target

</candidate_a_patch>

<candidate_b_patch>
diff --git a/django/db/models/fields/related.py b/django/db/models/fields/related.py
--- a/django/db/models/fields/related.py
+++ b/django/db/models/fields/related.py
@@ -582,7 +582,11 @@ def deconstruct(self):
         if self.remote_field.parent_link:
             kwargs['parent_link'] = self.remote_field.parent_link
         if isinstance(self.remote_field.model, str):
-            kwargs['to'] = self.remote_field.model.lower()
+            if '.' in self.remote_field.model:
+                app_label, model_name = self.remote_field.model.split('.')
+                kwargs['to'] = '%s.%s' % (app_label, model_name.lower())
+            else:
+                kwargs['to'] = self.remote_field.model.lower()
         else:
             kwargs['to'] = self.remote_field.model._meta.label_lower
         # If swappable is True, then see if we're actually pointing to the target

</candidate_b_patch>

<generated_test_proposal>
{
  "test_name": "test_foreign_key_deconstruct_dotted_mixed_case_app_label",
  "specification_gap": "ForeignKey deconstruction must preserve the complete, case-sensitive app label while normalizing only the model name, including when a valid app label contains dots. Its serialized output must remain stable when reconstructed through Field.clone().",
  "input_description": "Create a model with app_label='package.MixedCaseApp', create a ForeignKey to its class, clone the field so the relation becomes its serialized string form, and deconstruct the clone.",
  "expected_output": "The reconstructed field deconstructs with kwargs['to'] equal to 'package.MixedCaseApp.target'. Candidate B instead raises ValueError because split('.') produces three components.",
  "expected_passing_candidate": "candidate_a",
  "why_interesting": "This combines a supported dotted app-label boundary with the general field deconstruction/reconstruction invariant. Candidate A correctly treats the final dot as the model separator; candidate B assumes the entire reference contains exactly one dot.",
  "test_patch": "diff --git a/tests/field_deconstruction/tests.py b/tests/field_deconstruction/tests.py\n--- a/tests/field_deconstruction/tests.py\n+++ b/tests/field_deconstruction/tests.py\n@@ -1,7 +1,7 @@\n from django.apps import apps\n from django.db import models\n from django.test import SimpleTestCase, override_settings\n-from django.test.utils import isolate_lru_cache\n+from django.test.utils import isolate_apps, isolate_lru_cache\n \n \n class FieldDeconstructionTests(SimpleTestCase):\n@@ -193,6 +193,17 @@ class FieldDeconstructionTests(SimpleTestCase):\n         self.assertEqual(args, [])\n         self.assertEqual(kwargs, {})\n \n+    @isolate_apps()\n+    def test_foreign_key_deconstruct_dotted_mixed_case_app_label(self):\n+        class Target(models.Model):\n+            class Meta:\n+                app_label = 'package.MixedCaseApp'\n+\n+        field = models.ForeignKey(Target, models.CASCADE)\n+        reconstructed = field.clone()\n+        _, _, _, kwargs = reconstructed.deconstruct()\n+        self.assertEqual(kwargs['to'], 'package.MixedCaseApp.target')\n+\n     def test_foreign_key(self):\n         # Test basic pointing\n         from django.contrib.auth.models import Permission\n",
  "test_command": "python tests/runtests.py field_deconstruction.tests.FieldDeconstructionTests.test_foreign_key_deconstruct_dotted_mixed_case_app_label"
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
.
----------------------------------------------------------------------
Ran 1 test in 0.001s

OK
Testing against Django installed in '/testbed/django' with up to 24 processes
System check identified no issues (0 silenced).
[pipeline] test_exit_code=0

```

## candidate_b
returncode=1 passed=False test_ran=True
log=02_execution/attempt_02/candidate_b.log
```
[pipeline] checking candidate patch
[pipeline] checking generated test patch
[pipeline] executing generated test command
Testing against Django installed in '/testbed/django' with up to 24 processes
System check identified no issues (0 silenced).
E
======================================================================
ERROR: test_foreign_key_deconstruct_dotted_mixed_case_app_label (field_deconstruction.tests.FieldDeconstructionTests)
----------------------------------------------------------------------
Traceback (most recent call last):
  File "/testbed/django/test/utils.py", line 381, in inner
    return func(*args, **kwargs)
  File "/testbed/tests/field_deconstruction/tests.py", line 204, in test_foreign_key_deconstruct_dotted_mixed_case_app_label
    _, _, _, kwargs = reconstructed.deconstruct()
  File "/testbed/django/db/models/fields/related.py", line 875, in deconstruct
    name, path, args, kwargs = super().deconstruct()
  File "/testbed/django/db/models/fields/related.py", line 586, in deconstruct
    app_label, model_name = self.remote_field.model.split('.')
ValueError: too many values to unpack (expected 2)

----------------------------------------------------------------------
Ran 1 test in 0.001s

FAILED (errors=1)
[pipeline] test_exit_code=1

```
</validated_execution>
