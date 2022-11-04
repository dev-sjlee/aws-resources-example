---
description: Enable encrypt and decrypt using CMK to ElastiCache db instances.
---

# ElastiCache KMS Policies

## Granting encrypt and decrypt permissions to users or roles.

``` json linenums="1" hl_lines="5 11 15 16"
{
    "Sid": "Allow users or roles to use KMS to ElastiCache.",
    "Effect": "Allow",
    "Principal": {
        "AWS": "arn:aws:iam::<account id>:<users or roles>/<users or roles name>"
    },
    "Action": [
        "kms:CreateGrant",
        "kms:DescribeKey"
    ],
    "Resource": "arn:aws:kms:<region code>:<account id>:key/<kms key id>",
    "Condition": {
        "StringEquals": {
            "kms:ViaService": [
                "elasticache.<region code>.amazonaws.com",
                "dax.<region code>.amazonaws.com"
            ]
        }
    }
}
```

[AWS Documentation](https://docs.aws.amazon.com/AmazonElastiCache/latest/red-ug/at-rest-encryption.html)