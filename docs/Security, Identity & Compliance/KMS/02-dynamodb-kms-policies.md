---
description: Enable encrypt and decrypt using CMK to Dynamodb tables.
---

# DynamoDB KMS Policies

## Granting encrypt and decrypt permissions to users or roles.

``` json linenums="1" hl_lines="8 12 16"
{
    "Sid": "Allow users or roles to use KMS to DynamoDB.",
    "Effect": "Allow",
    "Principal": {
        "AWS": "arn:aws:iam::<account id>:<users or roles>/<users or roles name>"
    },
    "Action": [
        "kms:Decrypt"
    ],
    "Resource": "arn:aws:kms:<region code>:<account id>:key/<kms key id>"
}
```