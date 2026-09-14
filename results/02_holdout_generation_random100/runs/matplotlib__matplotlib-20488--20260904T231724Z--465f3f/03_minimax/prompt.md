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
test_huge_range_log is failing...
<!--To help us understand and resolve your issue, please fill out the form to the best of your ability.-->
<!--You can feel free to delete the sections that do not apply.-->

### Bug report

`lib/matplotlib/tests/test_image.py::test_huge_range_log` is failing quite a few of the CI runs with a Value Error.  

I cannot reproduce locally, so I assume there was a numpy change somewhere...

This test came in #18458


```
lib/matplotlib/image.py:638: in draw
    im, l, b, trans = self.make_image(
lib/matplotlib/image.py:924: in make_image
    return self._make_image(self._A, bbox, transformed_bbox, clip,
lib/matplotlib/image.py:542: in _make_image
    output = self.norm(resampled_masked)
_ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ 

self = <matplotlib.colors.LogNorm object at 0x7f057193f430>
value = masked_array(
  data=[[--, --, --, ..., --, --, --],
        [--, --, --, ..., --, --, --],
        [--, --, --, ..., ... False, False, ..., False, False, False],
        [False, False, False, ..., False, False, False]],
  fill_value=1e+20)
clip = False

    def __call__(self, value, clip=None):
        value, is_scalar = self.process_value(value)
        self.autoscale_None(value)
        if self.vmin > self.vmax:
            raise ValueError("vmin must be less or equal to vmax")
        if self.vmin == self.vmax:
            return np.full_like(value, 0)
        if clip is None:
            clip = self.clip
        if clip:
            value = np.clip(value, self.vmin, self.vmax)
        t_value = self._trf.transform(value).reshape(np.shape(value))
        t_vmin, t_vmax = self._trf.transform([self.vmin, self.vmax])
        if not np.isfinite([t_vmin, t_vmax]).all():
>           raise ValueError("Invalid vmin or vmax")
E           ValueError: Invalid vmin or vmax
lib/matplotlib/colors.py:1477: ValueError
```



</issue_statement>

<original_test_patch>
diff --git a/lib/matplotlib/tests/test_image.py b/lib/matplotlib/tests/test_image.py
--- a/lib/matplotlib/tests/test_image.py
+++ b/lib/matplotlib/tests/test_image.py
@@ -1233,23 +1233,24 @@ def test_imshow_quantitynd():
     fig.canvas.draw()
 
 
+@pytest.mark.parametrize('x', [-1, 1])
 @check_figures_equal(extensions=['png'])
-def test_huge_range_log(fig_test, fig_ref):
-    data = np.full((5, 5), -1, dtype=np.float64)
+def test_huge_range_log(fig_test, fig_ref, x):
+    # parametrize over bad lognorm -1 values and large range 1 -> 1e20
+    data = np.full((5, 5), x, dtype=np.float64)
     data[0:2, :] = 1E20
 
     ax = fig_test.subplots()
-    im = ax.imshow(data, norm=colors.LogNorm(vmin=100, vmax=data.max()),
-                   interpolation='nearest', cmap='viridis')
+    ax.imshow(data, norm=colors.LogNorm(vmin=1, vmax=data.max()),
+              interpolation='nearest', cmap='viridis')
 
-    data = np.full((5, 5), -1, dtype=np.float64)
+    data = np.full((5, 5), x, dtype=np.float64)
     data[0:2, :] = 1000
 
-    cmap = copy(plt.get_cmap('viridis'))
-    cmap.set_under('w')
     ax = fig_ref.subplots()
-    im = ax.imshow(data, norm=colors.Normalize(vmin=100, vmax=data.max()),
-                   interpolation='nearest', cmap=cmap)
+    cmap = plt.get_cmap('viridis').with_extremes(under='w')
+    ax.imshow(data, norm=colors.Normalize(vmin=1, vmax=data.max()),
+              interpolation='nearest', cmap=cmap)
 
 
 @check_figures_equal()

</original_test_patch>

<candidate_a_patch>
diff --git a/lib/matplotlib/image.py b/lib/matplotlib/image.py
index 95bb42d75..134343ec4 100644
--- a/lib/matplotlib/image.py
+++ b/lib/matplotlib/image.py
@@ -533,7 +533,7 @@ class _ImageBase(martist.Artist, cm.ScalarMappable):
                 # that may have moved input values in/out of range
                 s_vmin, s_vmax = vrange
                 if isinstance(self.norm, mcolors.LogNorm):
-                    if s_vmin < 0:
+                    if s_vmin <= 0:
                         s_vmin = max(s_vmin, np.finfo(scaled_dtype).eps)
                 with cbook._setattr_cm(self.norm,
                                        vmin=s_vmin,

</candidate_a_patch>

<candidate_b_patch>
diff --git a/lib/matplotlib/image.py b/lib/matplotlib/image.py
--- a/lib/matplotlib/image.py
+++ b/lib/matplotlib/image.py
@@ -532,9 +532,9 @@ def _make_image(self, A, in_bbox, out_bbox, clip_bbox, magnification=1.0,
                 # we have re-set the vmin/vmax to account for small errors
                 # that may have moved input values in/out of range
                 s_vmin, s_vmax = vrange
-                if isinstance(self.norm, mcolors.LogNorm):
-                    if s_vmin < 0:
-                        s_vmin = max(s_vmin, np.finfo(scaled_dtype).eps)
+                if isinstance(self.norm, mcolors.LogNorm) and s_vmin <= 0:
+                    # Don't give 0 or negative values to LogNorm
+                    s_vmin = np.finfo(scaled_dtype).eps
                 with cbook._setattr_cm(self.norm,
                                        vmin=s_vmin,
                                        vmax=s_vmax,

</candidate_b_patch>

<generated_test_proposal>
{
  "test_name": "test_huge_range_log_with_image_max_attribute",
  "specification_gap": "Rendering a logarithmically normalized image should use a fixed positive machine-epsilon floor when rescaling produces a nonpositive temporary lower bound; it should not depend on an unrelated `max` attribute in the public `matplotlib.image` module namespace.",
  "input_description": "Add `numpy.max` as `matplotlib.image.max`, then render a 5x5 float64 image containing values from -1 to 1e20 using `LogNorm(vmin=100, vmax=1e20)` and nearest-neighbor interpolation.",
  "expected_output": "The canvas draw completes without raising an exception. Candidate B directly assigns the positive epsilon floor, while candidate A resolves its unqualified `max(...)` through the newly present module attribute and calls `numpy.max` with epsilon as an invalid axis argument.",
  "expected_passing_candidate": "candidate_b",
  "why_interesting": "This exercises the public pyplot-to-canvas rendering path and verifies that handling a nonpositive temporary LogNorm bound is deterministic rather than dependent on mutable module namespace contents.",
  "test_patch": "diff --git a/lib/matplotlib/tests/test_image.py b/lib/matplotlib/tests/test_image.py\n--- a/lib/matplotlib/tests/test_image.py\n+++ b/lib/matplotlib/tests/test_image.py\n@@ -1251,6 +1251,17 @@ def test_huge_range_log(fig_test, fig_ref):\n     im = ax.imshow(data, norm=colors.Normalize(vmin=100, vmax=data.max()),\n                    interpolation='nearest', cmap=cmap)\n \n+\n+def test_huge_range_log_with_image_max_attribute(monkeypatch):\n+    monkeypatch.setattr(mimage, \"max\", np.max, raising=False)\n+    data = np.full((5, 5), -1, dtype=np.float64)\n+    data[:2] = 1e20\n+\n+    fig, ax = plt.subplots()\n+    ax.imshow(data, norm=colors.LogNorm(vmin=100, vmax=data.max()),\n+              interpolation=\"nearest\")\n+    fig.canvas.draw()\n+\n \n @check_figures_equal()\n def test_spy_box(fig_test, fig_ref):\n",
  "test_command": "python -m pytest -q lib/matplotlib/tests/test_image.py::test_huge_range_log_with_image_max_attribute"
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
F                                                                        [100%]
=================================== FAILURES ===================================
_________________ test_huge_range_log_with_image_max_attribute _________________

monkeypatch = <_pytest.monkeypatch.MonkeyPatch object at 0x75058dff79d0>

    def test_huge_range_log_with_image_max_attribute(monkeypatch):
        monkeypatch.setattr(mimage, "max", np.max, raising=False)
        data = np.full((5, 5), -1, dtype=np.float64)
        data[:2] = 1e20
    
        fig, ax = plt.subplots()
        ax.imshow(data, norm=colors.LogNorm(vmin=100, vmax=data.max()),
                  interpolation="nearest")
>       fig.canvas.draw()

lib/matplotlib/tests/test_image.py:1263: 
_ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ 
lib/matplotlib/backends/backend_agg.py:387: in draw
    self.figure.draw(self.renderer)
lib/matplotlib/artist.py:71: in draw_wrapper
    result = draw(artist, renderer, *args, **kwargs)
lib/matplotlib/artist.py:48: in draw_wrapper
    return draw(artist, renderer)
lib/matplotlib/figure.py:2754: in draw
    mimage._draw_list_compositing_images(
lib/matplotlib/image.py:132: in _draw_list_compositing_images
    a.draw(renderer)
lib/matplotlib/artist.py:48: in draw_wrapper
    return draw(artist, renderer)
lib/matplotlib/axes/_base.py:3074: in draw
    mimage._draw_list_compositing_images(
lib/matplotlib/image.py:132: in _draw_list_compositing_images
    a.draw(renderer)
lib/matplotlib/artist.py:48: in draw_wrapper
    return draw(artist, renderer)
lib/matplotlib/image.py:638: in draw
    im, l, b, trans = self.make_image(
lib/matplotlib/image.py:924: in make_image
    return self._make_image(self._A, bbox, transformed_bbox, clip,
lib/matplotlib/image.py:537: in _make_image
    s_vmin = max(s_vmin, np.finfo(scaled_dtype).eps)
<__array_function__ internals>:200: in amax
    ???
/opt/miniconda3/envs/testbed/lib/python3.8/site-packages/numpy/core/fromnumeric.py:2820: in amax
    return _wrapreduction(a, np.maximum, 'max', axis, None, out,
/opt/miniconda3/envs/testbed/lib/python3.8/site-packages/numpy/core/fromnumeric.py:84: in _wrapreduction
    return reduction(axis=axis, out=out, **passkwargs)
_ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ 

a = array(0.), axis = 2.220446049250313e-16, out = None, keepdims = False
initial = <no value>, where = True

    def _amax(a, axis=None, out=None, keepdims=False,
              initial=_NoValue, where=True):
>       return umr_maximum(a, axis, None, out, keepdims, initial, where)
E       TypeError: 'numpy.float64' object cannot be interpreted as an integer

/opt/miniconda3/envs/testbed/lib/python3.8/site-packages/numpy/core/_methods.py:41: TypeError
=========================== short test summary info ============================
FAILED lib/matplotlib/tests/test_image.py::test_huge_range_log_with_image_max_attribute
1 failed in 2.26s
[pipeline] test_exit_code=1

```

## candidate_b
returncode=0 passed=True test_ran=True
log=02_execution/attempt_02/candidate_b.log
```
[pipeline] checking candidate patch
[pipeline] checking generated test patch
[pipeline] executing generated test command
.                                                                        [100%]
1 passed in 1.95s
[pipeline] test_exit_code=0

```
</validated_execution>
