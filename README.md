# Four-round differentiating pipeline

This repository contains the results of the 100-instance experiment. The pipeline repeatedly generated candidate patches, evaluated them against an evolving test bank, and searched for behavioral tests that distinguished two otherwise-passing candidates.

## Start here

1. [`final_result.json`](final_result.json) lists all 100 instances, grouped by how they finished.
2. [`rounds/`](rounds/) tells the experiment round by round. Open a round and click an instance to see what happened and how its test bank changed.
3. [`test_bank/`](test_bank/) contains every generated test considered by the pipeline, with accepted and rejected tests clearly labeled.

## Final results

| How the instance finished | Count |
|---|---:|
| Candidate patches were identical | 43 |
| Enough separating evidence was collected | 14 |
| Reached the four-round limit | 20 |
| No generated patch passed the current bank | 10 |
| No additional distinct pair could be generated | 7 |
| Gold-check execution failure | 4 |
| Candidate-generation technical failure | 2 |

These are pipeline stopping reasons, not a simple correct/incorrect score.

## Test-bank language

- **Must pass:** the real gold patch passed the generated test, so later candidates also had to pass it.
- **Must fail:** the pipeline retained a specific behavioral failure as a constraint.
- **Rejected:** the test was considered but was not added to the active bank.

All four rounds ran and all 100 instances reached a stopping category. The original controller labeled the overall run `infrastructure_failed` because a separate final bank-validation step was not performed; that label does not mean the four rounds failed to run.
