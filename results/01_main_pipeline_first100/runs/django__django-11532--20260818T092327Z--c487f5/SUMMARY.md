# Differentiating-test run: `django__django-11532`

- Status: **completed**
- Image: `docker.io/swebench/sweb.eval.x86_64.django_1776_django-11532:latest`
- Run directory: `/home/rocky/SWE-ABS/mini-swe-agent/evaluation_records/differentiating_pipeline_vs_manual_first100_20260818/runs/django__django-11532--20260818T092327Z--c487f5`
- Test: `test_ascii_fqdn_is_not_idna_validated`
- Test command: `./tests/runtests.py mail.test_dns_name.CachedDnsNameTests.test_ascii_fqdn_is_not_idna_validated`

## Specification gap

IDNA conversion should fix non-ASCII hostnames without newly validating or rejecting hostnames that are already ASCII. CachedDnsName previously returned every ASCII socket.getfqdn() result unchanged.

## Input/output contract

Input: Mock socket.getfqdn() to return a single 64-character ASCII label and call CachedDnsName.get_fqdn() on a fresh cache instance.

Expected output: The exact 64-character hostname is returned unchanged without raising an exception.

## Execution

| Candidate | Ran | Passed | Exit |
| --- | --- | --- | --- |
| candidate_a | True | True | 0 |
| candidate_b | True | False | 1 |

## MiniMax judgment

- Rating: **high_signal**
- Confidence: 0.95
- Summary: The test successfully differentiates between two legitimate implementation approaches for handling non-ASCII hostnames in email Message-ID headers. Candidate A preserves ASCII hostnames unchanged (original behavior), while Candidate B unconditionally applies IDNA encoding which breaks valid ASCII hostnames exceeding 63 characters. The test correctly identifies this behavioral difference and has a defensible oracle based on the principle that the fix should only convert non-ASCII domains, not alter ASCII ones. Candidate A passes and appears more specification-conformant as it maintains backward compatibility.

## Main artifacts

- `selected_test.patch`
- `selected_proposal.json`
- `02_execution/selected/result.json`
- `03_minimax/verdict.json`
- `pipeline.log`
