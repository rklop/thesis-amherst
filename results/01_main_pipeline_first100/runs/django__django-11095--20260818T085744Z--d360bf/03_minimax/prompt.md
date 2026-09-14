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
add ModelAdmin.get_inlines() hook to allow set inlines based on the request or model instance.
Description
	
add ModelAdmin.get_inlines() hook to allow set inlines based on the request or model instance.
Currently, We can override the method get_inline_instances to do such a thing, but a for loop should be copied to my code. So I wished add a hook get_inlines(request, obj=None)

</issue_statement>

<original_test_patch>
diff --git a/tests/generic_inline_admin/tests.py b/tests/generic_inline_admin/tests.py
--- a/tests/generic_inline_admin/tests.py
+++ b/tests/generic_inline_admin/tests.py
@@ -429,3 +429,29 @@ class EpisodeAdmin(admin.ModelAdmin):
         inlines = ma.get_inline_instances(request)
         for (formset, inline), other_inline in zip(ma.get_formsets_with_inlines(request), inlines):
             self.assertIsInstance(formset, other_inline.get_formset(request).__class__)
+
+    def test_get_inline_instances_override_get_inlines(self):
+        class MediaInline(GenericTabularInline):
+            model = Media
+
+        class AlternateInline(GenericTabularInline):
+            model = Media
+
+        class EpisodeAdmin(admin.ModelAdmin):
+            inlines = (AlternateInline, MediaInline)
+
+            def get_inlines(self, request, obj):
+                if hasattr(request, 'name'):
+                    if request.name == 'alternate':
+                        return self.inlines[:1]
+                    elif request.name == 'media':
+                        return self.inlines[1:2]
+                return []
+
+        ma = EpisodeAdmin(Episode, self.site)
+        self.assertEqual(ma.get_inlines(request, None), [])
+        self.assertEqual(ma.get_inline_instances(request), [])
+        for name, inline_class in (('alternate', AlternateInline), ('media', MediaInline)):
+            request.name = name
+            self.assertEqual(ma.get_inlines(request, None), (inline_class,)),
+            self.assertEqual(type(ma.get_inline_instances(request)[0]), inline_class)

</original_test_patch>

<candidate_a_patch>
diff --git a/django/contrib/admin/options.py b/django/contrib/admin/options.py
index 5e7b23f9a0..a66237d64c 100644
--- a/django/contrib/admin/options.py
+++ b/django/contrib/admin/options.py
@@ -580,9 +580,15 @@ class ModelAdmin(BaseModelAdmin):
     def __str__(self):
         return "%s.%s" % (self.model._meta.app_label, self.__class__.__name__)
 
+    def get_inlines(self, request, obj=None):
+        """
+        Return the inlines to be used for this request.
+        """
+        return self.inlines
+
     def get_inline_instances(self, request, obj=None):
         inline_instances = []
-        for inline_class in self.inlines:
+        for inline_class in self.get_inlines(request, obj):
             inline = inline_class(self.model, self.admin_site)
             if request:
                 if not (inline.has_view_or_change_permission(request, obj) or

</candidate_a_patch>

<candidate_b_patch>
diff --git a/django/contrib/admin/options.py b/django/contrib/admin/options.py
--- a/django/contrib/admin/options.py
+++ b/django/contrib/admin/options.py
@@ -327,6 +327,10 @@ def get_fieldsets(self, request, obj=None):
             return self.fieldsets
         return [(None, {'fields': self.get_fields(request, obj)})]
 
+    def get_inlines(self, request, obj):
+        """Hook for specifying custom inlines."""
+        return self.inlines
+
     def get_ordering(self, request):
         """
         Hook for specifying field ordering.
@@ -582,7 +586,7 @@ def __str__(self):
 
     def get_inline_instances(self, request, obj=None):
         inline_instances = []
-        for inline_class in self.inlines:
+        for inline_class in self.get_inlines(request, obj):
             inline = inline_class(self.model, self.admin_site)
             if request:
                 if not (inline.has_view_or_change_permission(request, obj) or

</candidate_b_patch>

<generated_test_proposal>
{
  "test_name": "test_get_inlines_uses_default_obj",
  "specification_gap": "The issue specifies the public hook as get_inlines(request, obj=None), but existing coverage always supplies obj explicitly or reaches it through get_inline_instances(), which forwards obj. It therefore misses whether obj is truly optional.",
  "input_description": "Configure an Episode ModelAdmin with MediaInline and call get_inlines(request) without an obj argument, representing use when no model instance exists, such as an add view.",
  "expected_output": "The call returns the configured [MediaInline] list without raising TypeError.",
  "expected_passing_candidate": "candidate_a",
  "why_interesting": "This exercises an alternate public entry point and a meaningful arity boundary. Candidate A provides the specified obj=None default; candidate B requires obj even though the issue explicitly describes it as optional.",
  "test_patch": "diff --git a/tests/generic_inline_admin/tests.py b/tests/generic_inline_admin/tests.py\n--- a/tests/generic_inline_admin/tests.py\n+++ b/tests/generic_inline_admin/tests.py\n@@ -429,3 +429,10 @@ class GenericInlineModelAdminTest(SimpleTestCase):\n         inlines = ma.get_inline_instances(request)\n         for (formset, inline), other_inline in zip(ma.get_formsets_with_inlines(request), inlines):\n             self.assertIsInstance(formset, other_inline.get_formset(request).__class__)\n+\n+    def test_get_inlines_uses_default_obj(self):\n+        class EpisodeAdmin(admin.ModelAdmin):\n+            inlines = [MediaInline]\n+\n+        ma = EpisodeAdmin(Episode, self.site)\n+        self.assertEqual(ma.get_inlines(request), [MediaInline])\n",
  "test_command": "cd /testbed && python tests/runtests.py generic_inline_admin.tests.GenericInlineModelAdminTest.test_get_inlines_uses_default_obj"
}
</generated_test_proposal>

<validated_execution>
## candidate_a
returncode=0 passed=True test_ran=True
log=02_execution/attempt_01/candidate_a.log
```
[pipeline] checking candidate patch
[pipeline] checking generated test patch
[pipeline] executing generated test command
.
----------------------------------------------------------------------
Ran 1 test in 0.000s

OK
Testing against Django installed in '/testbed/django' with up to 24 processes
System check identified no issues (0 silenced).
[pipeline] test_exit_code=0

```

## candidate_b
returncode=1 passed=False test_ran=True
log=02_execution/attempt_01/candidate_b.log
```
[pipeline] checking candidate patch
[pipeline] checking generated test patch
[pipeline] executing generated test command
E
======================================================================
ERROR: test_get_inlines_uses_default_obj (generic_inline_admin.tests.GenericInlineModelAdminTest)
----------------------------------------------------------------------
Traceback (most recent call last):
  File "/testbed/tests/generic_inline_admin/tests.py", line 438, in test_get_inlines_uses_default_obj
    self.assertEqual(ma.get_inlines(request), [MediaInline])
TypeError: get_inlines() missing 1 required positional argument: 'obj'

----------------------------------------------------------------------
Ran 1 test in 0.000s

FAILED (errors=1)
Testing against Django installed in '/testbed/django' with up to 24 processes
System check identified no issues (0 silenced).
[pipeline] test_exit_code=1

```
</validated_execution>
