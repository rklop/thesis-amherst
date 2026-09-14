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
Subclassed SkyCoord gives misleading attribute access message
I'm trying to subclass `SkyCoord`, and add some custom properties. This all seems to be working fine, but when I have a custom property (`prop` below) that tries to access a non-existent attribute (`random_attr`) below, the error message is misleading because it says `prop` doesn't exist, where it should say `random_attr` doesn't exist.

```python
import astropy.coordinates as coord


class custom_coord(coord.SkyCoord):
    @property
    def prop(self):
        return self.random_attr


c = custom_coord('00h42m30s', '+41d12m00s', frame='icrs')
c.prop
```

raises
```
Traceback (most recent call last):
  File "test.py", line 11, in <module>
    c.prop
  File "/Users/dstansby/miniconda3/lib/python3.7/site-packages/astropy/coordinates/sky_coordinate.py", line 600, in __getattr__
    .format(self.__class__.__name__, attr))
AttributeError: 'custom_coord' object has no attribute 'prop'
```

</issue_statement>

<original_test_patch>
diff --git a/astropy/coordinates/tests/test_sky_coord.py b/astropy/coordinates/tests/test_sky_coord.py
--- a/astropy/coordinates/tests/test_sky_coord.py
+++ b/astropy/coordinates/tests/test_sky_coord.py
@@ -2165,3 +2165,21 @@ def test_match_to_catalog_3d_and_sky():
     npt.assert_array_equal(idx, [0, 1, 2, 3])
     assert_allclose(angle, 0 * u.deg, atol=1e-14 * u.deg, rtol=0)
     assert_allclose(distance, 0 * u.kpc, atol=1e-14 * u.kpc, rtol=0)
+
+
+def test_subclass_property_exception_error():
+    """Regression test for gh-8340.
+
+    Non-existing attribute access inside a property should give attribute
+    error for the attribute, not for the property.
+    """
+
+    class custom_coord(SkyCoord):
+        @property
+        def prop(self):
+            return self.random_attr
+
+    c = custom_coord("00h42m30s", "+41d12m00s", frame="icrs")
+    with pytest.raises(AttributeError, match="random_attr"):
+        # Before this matched "prop" rather than "random_attr"
+        c.prop

</original_test_patch>

<candidate_a_patch>
diff --git a/astropy/coordinates/sky_coordinate.py b/astropy/coordinates/sky_coordinate.py
index ab475f7d0..b8e148475 100644
--- a/astropy/coordinates/sky_coordinate.py
+++ b/astropy/coordinates/sky_coordinate.py
@@ -895,6 +895,18 @@ class SkyCoord(ShapedLikeNDArray):
                 return self.transform_to(attr)
 
         # Fail
+        # First check if attr is actually a property/descriptor on the class
+        # (e.g., from a subclass). If so, we should not mask the original error.
+        for klass in type(self).__mro__:
+            if attr in klass.__dict__:
+                descriptor = klass.__dict__[attr]
+                if hasattr(descriptor, '__get__'):
+                    # It's a descriptor (e.g., property). Calling it directly
+                    # will re-raise the original AttributeError from within
+                    # the descriptor.
+                    return descriptor.__get__(self, type(self))
+                break
+
         raise AttributeError(
             f"'{self.__class__.__name__}' object has no attribute '{attr}'"
         )

</candidate_a_patch>

<candidate_b_patch>
diff --git a/astropy/coordinates/sky_coordinate.py b/astropy/coordinates/sky_coordinate.py
--- a/astropy/coordinates/sky_coordinate.py
+++ b/astropy/coordinates/sky_coordinate.py
@@ -894,10 +894,8 @@ def __getattr__(self, attr):
             if frame_cls is not None and self.frame.is_transformable_to(frame_cls):
                 return self.transform_to(attr)
 
-        # Fail
-        raise AttributeError(
-            f"'{self.__class__.__name__}' object has no attribute '{attr}'"
-        )
+        # Call __getattribute__; this will give correct exception.
+        return self.__getattribute__(attr)
 
     def __setattr__(self, attr, val):
         # This is to make anything available through __getattr__ immutable

</candidate_b_patch>

<generated_test_proposal>
{
  "test_name": "test_subclass_property_preserves_attribute_error_type",
  "specification_gap": "SkyCoord subclass properties should preserve not only the nested AttributeError message, but also its public exception subtype when attribute lookup is customized by the subclass.",
  "input_description": "A SkyCoord subclass defines a property that accesses `random_attr`. Its public `__getattribute__` hook raises a custom `MissingAttribute` subtype for that name.",
  "expected_output": "Accessing `coord.prop` raises `MissingAttribute`, preserving the nested lookup failure instead of replacing it with a newly constructed base AttributeError.",
  "expected_passing_candidate": "candidate_b",
  "why_interesting": "Candidate A only retries descriptors and synthesizes a base AttributeError while processing `random_attr`, losing the original subtype. Candidate B delegates to normal `__getattribute__` lookup, which preserves the custom exception. This exercises subclassing through Python's public attribute protocol rather than checking implementation details or message formatting.",
  "test_patch": "diff --git a/astropy/coordinates/tests/test_sky_coord.py b/astropy/coordinates/tests/test_sky_coord.py\n--- a/astropy/coordinates/tests/test_sky_coord.py\n+++ b/astropy/coordinates/tests/test_sky_coord.py\n@@ -2165,3 +2165,23 @@ def test_match_to_catalog_3d_and_sky():\n     npt.assert_array_equal(idx, [0, 1, 2, 3])\n     assert_allclose(angle, 0 * u.deg, atol=1e-14 * u.deg, rtol=0)\n     assert_allclose(distance, 0 * u.kpc, atol=1e-14 * u.kpc, rtol=0)\n+\n+\n+def test_subclass_property_preserves_attribute_error_type():\n+    class MissingAttribute(AttributeError):\n+        pass\n+\n+    class CustomCoord(SkyCoord):\n+        @property\n+        def prop(self):\n+            return self.random_attr\n+\n+        def __getattribute__(self, attr):\n+            if attr == \"random_attr\":\n+                raise MissingAttribute(attr)\n+            return super().__getattribute__(attr)\n+\n+    coord = CustomCoord(1, 2, unit=\"deg\")\n+\n+    with pytest.raises(MissingAttribute):\n+        coord.prop\n",
  "test_command": "cd /testbed && python -m pytest -q astropy/coordinates/tests/test_sky_coord.py::test_subclass_property_preserves_attribute_error_type"
}
</generated_test_proposal>

<validated_execution>
## candidate_a
returncode=1 passed=False test_ran=True
log=02_execution/attempt_01/candidate_a.log
```
[pipeline] checking candidate patch
[pipeline] checking generated test patch
[pipeline] executing generated test command
<frozen importlib._bootstrap>:228: RuntimeWarning: numpy.ndarray size changed, may indicate binary incompatibility. Expected 80 from C header, got 96 from PyObject
Internet access disabled
[31mF[0m[31m                                                                        [100%][0m
=================================== FAILURES ===================================
[31m[1m____________ test_subclass_property_preserves_attribute_error_type _____________[0m

    def test_subclass_property_preserves_attribute_error_type():
        class MissingAttribute(AttributeError):
            pass
    
        class CustomCoord(SkyCoord):
            @property
            def prop(self):
                return self.random_attr
    
            def __getattribute__(self, attr):
                if attr == "random_attr":
                    raise MissingAttribute(attr)
                return super().__getattribute__(attr)
    
        coord = CustomCoord(1, 2, unit="deg")
    
        with pytest.raises(MissingAttribute):
>           coord.prop

[1m[31mastropy/coordinates/tests/test_sky_coord.py[0m:2187: 
_ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ 
[1m[31mastropy/coordinates/sky_coordinate.py[0m:907: in __getattr__
    return descriptor.__get__(self, type(self))
[1m[31mastropy/coordinates/tests/test_sky_coord.py[0m:2177: in prop
    return self.random_attr
_ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ 

self = <CustomCoord (ICRS): (ra, dec) in deg
    (1., 2.)>, attr = 'random_attr'

    def __getattr__(self, attr):
        """
        Overrides getattr to return coordinates that this can be transformed
        to, based on the alias attr in the primary transform graph.
        """
        if "_sky_coord_frame" in self.__dict__:
            if self._is_name(attr):
                return self  # Should this be a deepcopy of self?
    
            # Anything in the set of all possible frame_attr_names is handled
            # here. If the attr is relevant for the current frame then delegate
            # to self.frame otherwise get it from self._<attr>.
            if attr in frame_transform_graph.frame_attributes:
                if attr in self.frame.frame_attributes:
                    return getattr(self.frame, attr)
                else:
                    return getattr(self, "_" + attr, None)
    
            # Some attributes might not fall in the above category but still
            # are available through self._sky_coord_frame.
            if not attr.startswith("_") and hasattr(self._sky_coord_frame, attr):
                return getattr(self._sky_coord_frame, attr)
    
            # Try to interpret as a new frame for transforming.
            frame_cls = frame_transform_graph.lookup_name(attr)
            if frame_cls is not None and self.frame.is_transformable_to(frame_cls):
                return self.transform_to(attr)
    
        # Fail
        # First check if attr is actually a property/descriptor on the class
        # (e.g., from a subclass). If so, we should not mask the original error.
        for klass in type(self).__mro__:
            if attr in klass.__dict__:
                descriptor = klass.__dict__[attr]
                if hasattr(descriptor, '__get__'):
                    # It's a descriptor (e.g., property). Calling it directly
                    # will re-raise the original AttributeError from within
                    # the descriptor.
                    return descriptor.__get__(self, type(self))
                break
    
>       raise AttributeError(
            f"'{self.__class__.__name__}' object has no attribute '{attr}'"
        )
[1m[31mE       AttributeError: 'CustomCoord' object has no attribute 'random_attr'[0m

[1m[31mastropy/coordinates/sky_coordinate.py[0m:910: AttributeError
[36m[1m=========================== short test summary info ============================[0m
[31mFAILED[0m astropy/coordinates/tests/test_sky_coord.py::[1mtest_subclass_property_preserves_attribute_error_type[0m - AttributeError: 'CustomCoord' object has no attribute 'random_attr'
[31m[31m[1m1 failed[0m[31m in 0.41s[0m[0m
[pipeline] test_exit_code=1

```

## candidate_b
returncode=0 passed=True test_ran=True
log=02_execution/attempt_01/candidate_b.log
```
[pipeline] checking candidate patch
[pipeline] checking generated test patch
[pipeline] executing generated test command
<frozen importlib._bootstrap>:228: RuntimeWarning: numpy.ndarray size changed, may indicate binary incompatibility. Expected 80 from C header, got 96 from PyObject
Internet access disabled
[32m.[0m[32m                                                                        [100%][0m
[32m[32m[1m1 passed[0m[32m in 0.34s[0m[0m
[pipeline] test_exit_code=0

```
</validated_execution>
