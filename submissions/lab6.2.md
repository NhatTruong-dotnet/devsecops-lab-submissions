# Paste the full contents of:
[policies](https://github.com/NhatTruong-dotnet/devsecops-lab-submissions/tree/feature/lab6.2/scripts/policies)

### Rule fires
The command returned two failed checks for CKV2_CUSTOM_1:
```
[
  {
    "check_id": "CKV2_CUSTOM_1",
    "check_name": "Ensure RDS instances enable IAM database authentication",
    "check_result": {
      "result": "FAILED",
      "evaluated_keys": [
        "iam_database_authentication_enabled",
        "resource_type"
      ]
    },
    "resource": "aws_db_instance.unencrypted_db",
    "severity": "HIGH"
  },
  {
    "check_id": "CKV2_CUSTOM_1",
    "check_name": "Ensure RDS instances enable IAM database authentication",
    "check_result": {
      "result": "FAILED",
      "evaluated_keys": [
        "iam_database_authentication_enabled",
        "resource_type"
      ]
    },
    "resource": "aws_db_instance.weak_db",
    "severity": "HIGH"
  }
]
```

This confirms that the custom rule ID CKV2_CUSTOM_1 is being detected by Checkov and is correctly identifying RDS resources that do not satisfy the policy requirement.

Why this rule matters

Enforcing IAM database authentication for Amazon RDS reduces reliance on long-lived database passwords and supports stronger, centrally managed authentication and access control. This is particularly important for preventing credential exposure and improving identity-based access management in cloud environments, aligning with security principles such as least privilege and NIST SP 800-53 access-control requirements.
