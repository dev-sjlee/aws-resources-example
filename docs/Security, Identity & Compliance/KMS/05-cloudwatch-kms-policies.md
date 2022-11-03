---
description: Enable encrypt and decrypt using CMK to CloudWatch log groups.
---

# CloudWatch Logs KMS Policies

## Granting encrypt and decrypt permissions to CloudWatch Logs

``` json linenums="1" hl_lines="4 16"
{
    "Effect": "Allow",
    "Principal": {
        "Service": "logs.<region code>.amazonaws.com"
    },
    "Action": [
            "kms:Encrypt*",
            "kms:Decrypt*",
            "kms:ReEncrypt*",
            "kms:GenerateDataKey*",
            "kms:Describe*"
    ],
    "Resource": "*",
    "Condition": {
        "ArnEquals": {
            "kms:EncryptionContext:aws:logs:arn": "arn:aws:logs:<region code>:<account id>:log-group:<log group name>"
        }
    }
}
```

You can use `arn:aws:logs:<region code>:<account id>:log-group:*` at `Condition`.

[AWS Documentation](https://docs.aws.amazon.com/AmazonCloudWatch/latest/logs/encrypt-log-data-kms.html)