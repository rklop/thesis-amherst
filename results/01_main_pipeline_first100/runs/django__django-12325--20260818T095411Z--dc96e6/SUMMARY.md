# Differentiating-test run: `django__django-12325`

- Status: **completed**
- Image: `docker.io/swebench/sweb.eval.x86_64.django_1776_django-12325:latest`
- Run directory: `/home/rocky/SWE-ABS/mini-swe-agent/evaluation_records/differentiating_pipeline_vs_manual_first100_20260818/runs/django__django-12325--20260818T095411Z--dc96e6`
- Test: `test_abstract_onetoone_named_as_parent_pointer`
- Test command: `PYTHONPATH=. python tests/runtests.py invalid_models_tests.test_abstract_parent_link.ParentLinkTests.test_abstract_onetoone_named_as_parent_pointer`

## Specification gap

An unrelated OneToOneField inherited from an abstract base must not be silently repurposed as the concrete model's multi-table-inheritance parent link merely because its name matches the generated parent pointer. Such an ambiguous model definition must be rejected unless the field explicitly declares parent_link=True.

## Input/output contract

Input: Define an abstract model containing place_ptr = OneToOneField(Place) without parent_link=True, then define ParkingLot(AbstractParkingLot, Place).

Expected output: Defining ParkingLot raises ImproperlyConfigured. It must not construct a model whose unrelated OneToOneField is implicitly promoted to the inheritance link.

## Execution

| Candidate | Ran | Passed | Exit |
| --- | --- | --- | --- |
| candidate_a | True | True | 0 |
| candidate_b | True | False | 1 |

## MiniMax judgment

- Rating: **high_signal**
- Confidence: 0.9
- Summary: The test exercises a valid public behavior: Django should raise ImproperlyConfigured when an abstract model's OneToOneField could be confused for an MTI parent link, requiring explicit parent_link=True. Candidate_a passes because it preserves this validation; candidate_b fails because it removes the check entirely, allowing silent misconfiguration. The test targets the specification gap in abstract model inheritance that wasn't covered by the original issue or patch.

## Main artifacts

- `selected_test.patch`
- `selected_proposal.json`
- `02_execution/selected/result.json`
- `03_minimax/verdict.json`
- `pipeline.log`
