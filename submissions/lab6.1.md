# Lab 6.1 — Submission

 ## Task 1: Checkov on Terraform

 ### Terraform scan (passed/failed per framework)

 | Framework | Passed | Failed |
| --- | --- | --- |
| terraform | 49 | 78 |
| secrets | 0 | 2 |

 ### Top 5 rule IDs (by frequency)

 | Rule ID | Count | What it checks |
| --- | --- | --- |
| CKV\_AWS\_289 | 4 | Checks that IAM policies do not allow resource exposure without restrictions |
| CKV\_AWS\_355 | 4 | Checks that IAM policies do not use `*` as the resource for actions that can be restricted |
| CKV\_AWS\_23 | 3 | Checks that security group rules have descriptions |
| CKV\_AWS\_288 | 3 | Checks that IAM policies do not allow data exfiltration |
| CKV\_AWS\_290 | 3 | Checks that IAM policies do not allow write access without restrictions |

 ### Module-leverage analysis (Lecture 6 slide 17)

 The best fix would be to make the shared IAM policy more specific instead of using `Resource: "*"`. This could fix several IAM security findings at the same time because the same policy is used in multiple places.
