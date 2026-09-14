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
Admin inlines for auto-created ManyToManyFields are editable if the user only has the view permission
Description
	
From https://code.djangoproject.com/ticket/8060#comment:34
Replying to Will Gordon:
This seems to have regressed in (at least) 2.1. I have 2 view only permissions. I have a ManyToManyField represented in my main model as a TabularInline. But, my user with view only permissions can now add or remove these items at will!
I am having the same issue, so I assume this is a bug. I did not find Will had created a separate ticket.
models.py:
class Photo(models.Model):
	pass
class Report(models.Model):
	photos = models.ManyToManyField(Photo)
admin.py:
		class ReportPhotoInlineModelAdmin(admin.TabularInline):
			model = Report.photos.through
			show_change_link = True

</issue_statement>

<original_test_patch>
diff --git a/tests/admin_inlines/models.py b/tests/admin_inlines/models.py
--- a/tests/admin_inlines/models.py
+++ b/tests/admin_inlines/models.py
@@ -37,6 +37,9 @@ def __str__(self):
 class Book(models.Model):
     name = models.CharField(max_length=50)
 
+    def __str__(self):
+        return self.name
+
 
 class Author(models.Model):
     name = models.CharField(max_length=50)
diff --git a/tests/admin_inlines/tests.py b/tests/admin_inlines/tests.py
--- a/tests/admin_inlines/tests.py
+++ b/tests/admin_inlines/tests.py
@@ -595,10 +595,10 @@ def setUpTestData(cls):
         cls.user.user_permissions.add(permission)
 
         author = Author.objects.create(pk=1, name='The Author')
-        book = author.books.create(name='The inline Book')
+        cls.book = author.books.create(name='The inline Book')
         cls.author_change_url = reverse('admin:admin_inlines_author_change', args=(author.id,))
         # Get the ID of the automatically created intermediate model for the Author-Book m2m
-        author_book_auto_m2m_intermediate = Author.books.through.objects.get(author=author, book=book)
+        author_book_auto_m2m_intermediate = Author.books.through.objects.get(author=author, book=cls.book)
         cls.author_book_auto_m2m_intermediate_id = author_book_auto_m2m_intermediate.pk
 
         cls.holder = Holder2.objects.create(dummy=13)
@@ -636,6 +636,25 @@ def test_inline_change_fk_noperm(self):
         self.assertNotContains(response, 'Add another Inner2')
         self.assertNotContains(response, 'id="id_inner2_set-TOTAL_FORMS"')
 
+    def test_inline_add_m2m_view_only_perm(self):
+        permission = Permission.objects.get(codename='view_book', content_type=self.book_ct)
+        self.user.user_permissions.add(permission)
+        response = self.client.get(reverse('admin:admin_inlines_author_add'))
+        # View-only inlines. (It could be nicer to hide the empty, non-editable
+        # inlines on the add page.)
+        self.assertIs(response.context['inline_admin_formset'].has_view_permission, True)
+        self.assertIs(response.context['inline_admin_formset'].has_add_permission, False)
+        self.assertIs(response.context['inline_admin_formset'].has_change_permission, False)
+        self.assertIs(response.context['inline_admin_formset'].has_delete_permission, False)
+        self.assertContains(response, '<h2>Author-book relationships</h2>')
+        self.assertContains(
+            response,
+            '<input type="hidden" name="Author_books-TOTAL_FORMS" value="0" '
+            'id="id_Author_books-TOTAL_FORMS">',
+            html=True,
+        )
+        self.assertNotContains(response, 'Add another Author-Book Relationship')
+
     def test_inline_add_m2m_add_perm(self):
         permission = Permission.objects.get(codename='add_book', content_type=self.book_ct)
         self.user.user_permissions.add(permission)
@@ -665,11 +684,39 @@ def test_inline_change_m2m_add_perm(self):
         self.assertNotContains(response, 'id="id_Author_books-TOTAL_FORMS"')
         self.assertNotContains(response, 'id="id_Author_books-0-DELETE"')
 
+    def test_inline_change_m2m_view_only_perm(self):
+        permission = Permission.objects.get(codename='view_book', content_type=self.book_ct)
+        self.user.user_permissions.add(permission)
+        response = self.client.get(self.author_change_url)
+        # View-only inlines.
+        self.assertIs(response.context['inline_admin_formset'].has_view_permission, True)
+        self.assertIs(response.context['inline_admin_formset'].has_add_permission, False)
+        self.assertIs(response.context['inline_admin_formset'].has_change_permission, False)
+        self.assertIs(response.context['inline_admin_formset'].has_delete_permission, False)
+        self.assertContains(response, '<h2>Author-book relationships</h2>')
+        self.assertContains(
+            response,
+            '<input type="hidden" name="Author_books-TOTAL_FORMS" value="1" '
+            'id="id_Author_books-TOTAL_FORMS">',
+            html=True,
+        )
+        # The field in the inline is read-only.
+        self.assertContains(response, '<p>%s</p>' % self.book)
+        self.assertNotContains(
+            response,
+            '<input type="checkbox" name="Author_books-0-DELETE" id="id_Author_books-0-DELETE">',
+            html=True,
+        )
+
     def test_inline_change_m2m_change_perm(self):
         permission = Permission.objects.get(codename='change_book', content_type=self.book_ct)
         self.user.user_permissions.add(permission)
         response = self.client.get(self.author_change_url)
         # We have change perm on books, so we can add/change/delete inlines
+        self.assertIs(response.context['inline_admin_formset'].has_view_permission, True)
+        self.assertIs(response.context['inline_admin_formset'].has_add_permission, True)
+        self.assertIs(response.context['inline_admin_formset'].has_change_permission, True)
+        self.assertIs(response.context['inline_admin_formset'].has_delete_permission, True)
         self.assertContains(response, '<h2>Author-book relationships</h2>')
         self.assertContains(response, 'Add another Author-book relationship')
         self.assertContains(response, '<input type="hidden" id="id_Author_books-TOTAL_FORMS" '

</original_test_patch>

<candidate_a_patch>
diff --git a/django/contrib/admin/options.py b/django/contrib/admin/options.py
index 880a2cc17b..bfb68484ce 100644
--- a/django/contrib/admin/options.py
+++ b/django/contrib/admin/options.py
@@ -2115,29 +2115,43 @@ class InlineModelAdmin(BaseModelAdmin):
         if self.opts.auto_created:
             # We're checking the rights to an auto-created intermediate model,
             # which doesn't have its own individual permissions. The user needs
-            # to have the view permission for the related model in order to
-            # be able to do anything with the intermediate model.
-            return self.has_view_permission(request, obj)
+            # to have the change permission for the related model in order to
+            # be able to add/change/delete the intermediate model.
+            return self._has_change_permission(request, obj)
         return super().has_add_permission(request)
 
     def has_change_permission(self, request, obj=None):
         if self.opts.auto_created:
             # We're checking the rights to an auto-created intermediate model,
             # which doesn't have its own individual permissions. The user needs
-            # to have the view permission for the related model in order to
-            # be able to do anything with the intermediate model.
-            return self.has_view_permission(request, obj)
+            # to have the change permission for the related model in order to
+            # be able to add/change/delete the intermediate model.
+            return self._has_change_permission(request, obj)
         return super().has_change_permission(request)
 
     def has_delete_permission(self, request, obj=None):
         if self.opts.auto_created:
             # We're checking the rights to an auto-created intermediate model,
             # which doesn't have its own individual permissions. The user needs
-            # to have the view permission for the related model in order to
-            # be able to do anything with the intermediate model.
-            return self.has_view_permission(request, obj)
+            # to have the change permission for the related model in order to
+            # be able to add/change/delete the intermediate model.
+            return self._has_change_permission(request, obj)
         return super().has_delete_permission(request, obj)
 
+    def _has_change_permission(self, request, obj):
+        """
+        Return True if the user has change permission on the related model
+        for auto-created intermediate models.
+        """
+        opts = self.opts
+        # The model was auto-created as intermediary for a many-to-many
+        # Many-relationship; find the target model.
+        for field in opts.fields:
+            if field.remote_field and field.remote_field.model != self.parent_model:
+                opts = field.remote_field.model._meta
+                break
+        return request.user.has_perm('%s.%s' % (opts.app_label, get_permission_codename('change', opts)))
+
     def has_view_permission(self, request, obj=None):
         if self.opts.auto_created:
             opts = self.opts

</candidate_a_patch>

<candidate_b_patch>
diff --git a/django/contrib/admin/options.py b/django/contrib/admin/options.py
--- a/django/contrib/admin/options.py
+++ b/django/contrib/admin/options.py
@@ -2111,46 +2111,50 @@ def get_queryset(self, request):
             queryset = queryset.none()
         return queryset
 
+    def _has_any_perms_for_target_model(self, request, perms):
+        """
+        This method is called only when the ModelAdmin's model is for an
+        ManyToManyField's implicit through model (if self.opts.auto_created).
+        Return True if the user has any of the given permissions ('add',
+        'change', etc.) for the model that points to the through model.
+        """
+        opts = self.opts
+        # Find the target model of an auto-created many-to-many relationship.
+        for field in opts.fields:
+            if field.remote_field and field.remote_field.model != self.parent_model:
+                opts = field.remote_field.model._meta
+                break
+        return any(
+            request.user.has_perm('%s.%s' % (opts.app_label, get_permission_codename(perm, opts)))
+            for perm in perms
+        )
+
     def has_add_permission(self, request, obj):
         if self.opts.auto_created:
-            # We're checking the rights to an auto-created intermediate model,
-            # which doesn't have its own individual permissions. The user needs
-            # to have the view permission for the related model in order to
-            # be able to do anything with the intermediate model.
-            return self.has_view_permission(request, obj)
+            # Auto-created intermediate models don't have their own
+            # permissions. The user needs to have the change permission for the
+            # related model in order to be able to do anything with the
+            # intermediate model.
+            return self._has_any_perms_for_target_model(request, ['change'])
         return super().has_add_permission(request)
 
     def has_change_permission(self, request, obj=None):
         if self.opts.auto_created:
-            # We're checking the rights to an auto-created intermediate model,
-            # which doesn't have its own individual permissions. The user needs
-            # to have the view permission for the related model in order to
-            # be able to do anything with the intermediate model.
-            return self.has_view_permission(request, obj)
+            # Same comment as has_add_permission().
+            return self._has_any_perms_for_target_model(request, ['change'])
         return super().has_change_permission(request)
 
     def has_delete_permission(self, request, obj=None):
         if self.opts.auto_created:
-            # We're checking the rights to an auto-created intermediate model,
-            # which doesn't have its own individual permissions. The user needs
-            # to have the view permission for the related model in order to
-            # be able to do anything with the intermediate model.
-            return self.has_view_permission(request, obj)
+            # Same comment as has_add_permission().
+            return self._has_any_perms_for_target_model(request, ['change'])
         return super().has_delete_permission(request, obj)
 
     def has_view_permission(self, request, obj=None):
         if self.opts.auto_created:
-            opts = self.opts
-            # The model was auto-created as intermediary for a many-to-many
-            # Many-relationship; find the target model.
-            for field in opts.fields:
-                if field.remote_field and field.remote_field.model != self.parent_model:
-                    opts = field.remote_field.model._meta
-                    break
-            return (
-                request.user.has_perm('%s.%s' % (opts.app_label, get_permission_codename('view', opts))) or
-                request.user.has_perm('%s.%s' % (opts.app_label, get_permission_codename('change', opts)))
-            )
+            # Same comment as has_add_permission(). The 'change' permission
+            # also implies the 'view' permission.
+            return self._has_any_perms_for_target_model(request, ['view', 'change'])
         return super().has_view_permission(request)
 
 

</candidate_b_patch>

<generated_test_proposal>
{
  "test_name": "test_inline_change_m2m_private_method_name_collision",
  "specification_gap": "Fixing auto-created many-to-many permissions must preserve compatibility with existing InlineModelAdmin subclasses. A subclass-private method that Django previously ignored must not unexpectedly become an internal permission hook.",
  "input_description": "Create a TabularInline for Author.books.through whose subclass already defines `_has_change_permission(self, request)`. Grant the user change_book permission, then call the public `has_change_permission(request)` entry point.",
  "expected_output": "The permission check returns True without invoking the unrelated subclass-private method. Candidate A instead dispatches to that method with an additional obj argument and raises TypeError.",
  "expected_passing_candidate": "candidate_b",
  "why_interesting": "Both candidates handle ordinary view-only users similarly. This boundary case exposes candidate A's backward-incompatible helper-name collision through a public permission API, while candidate B uses a narrowly scoped helper name.",
  "test_patch": "diff --git a/tests/admin_inlines/tests.py b/tests/admin_inlines/tests.py\nindex e08e7509de..bf85aabb01 100644\n--- a/tests/admin_inlines/tests.py\n+++ b/tests/admin_inlines/tests.py\n@@ -608,6 +608,20 @@ class TestInlinePermissions(TestCase):\n         self.holder_change_url = reverse('admin:admin_inlines_holder2_change', args=(self.holder.id,))\n         self.client.force_login(self.user)\n \n+    def test_inline_change_m2m_private_method_name_collision(self):\n+        class CustomBookInline(TabularInline):\n+            model = Author.books.through\n+\n+            def _has_change_permission(self, request):\n+                return False\n+\n+        permission = Permission.objects.get(codename='change_book', content_type=self.book_ct)\n+        self.user.user_permissions.add(permission)\n+        request = RequestFactory().get('/')\n+        request.user = self.user\n+        inline = CustomBookInline(Author, admin_site)\n+        self.assertIs(inline.has_change_permission(request), True)\n+\n     def test_inline_add_m2m_noperm(self):\n         response = self.client.get(reverse('admin:admin_inlines_author_add'))\n         # No change permission on books, so no inline\n",
  "test_command": "python tests/runtests.py admin_inlines.tests.TestInlinePermissions.test_inline_change_m2m_private_method_name_collision"
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
E
======================================================================
ERROR: test_inline_change_m2m_private_method_name_collision (admin_inlines.tests.TestInlinePermissions)
----------------------------------------------------------------------
Traceback (most recent call last):
  File "/testbed/tests/admin_inlines/tests.py", line 623, in test_inline_change_m2m_private_method_name_collision
    self.assertIs(inline.has_change_permission(request), True)
  File "/testbed/django/contrib/admin/options.py", line 2129, in has_change_permission
    return self._has_change_permission(request, obj)
TypeError: _has_change_permission() takes 2 positional arguments but 3 were given

----------------------------------------------------------------------
Ran 1 test in 0.031s

FAILED (errors=1)
Destroying test database for alias 'default'...
Testing against Django installed in '/testbed/django' with up to 24 processes
System check identified no issues (0 silenced).
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
.
----------------------------------------------------------------------
Ran 1 test in 0.031s

OK
Destroying test database for alias 'default'...
Testing against Django installed in '/testbed/django' with up to 24 processes
System check identified no issues (0 silenced).
[pipeline] test_exit_code=0

```
</validated_execution>
