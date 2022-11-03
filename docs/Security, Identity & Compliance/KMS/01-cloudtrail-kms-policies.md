---
description: Enable encrypt and decrypt using CMK to CloudTrail trails.
---

# CloudTrail KMS Policies

1. Enable CloudTrail log encrypt permissions. See [Granting encrypt permissions](#granting-encrypt-permissions).
2. Enable CloudTrail log decrypt permissions. See [Grantind decrypt permissions](#granting-decrypt-permissions).
3. Enable CloudTrail to describe KMS key properties. See [Enable CloudTrail to describe KMS key properties](#enable-cloudtrail-to-describe-kms-key-properties).

[AWS Documentation](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/create-kms-key-policy-for-cloudtrail.html)

## Granting encrypt permissions

``` json linenums="1" hl_lines="8 12 16"
{
    "Sid": "Allow CloudTrail to encrypt logs",
    "Effect": "Allow",
    "Principal": {
        "Service": "cloudtrail.amazonaws.com"
    },
    "Action": "kms:GenerateDataKey*",
    "Resource": "arn:aws:kms:<region code>:<account id>:key/<kms key id>",
    "Condition": {
        "StringLike": {
            "kms:EncryptionContext:aws:cloudtrail:arn": [
                "arn:aws:cloudtrail:*:<account id>:trail/*"
            ]
        },
        "StringEquals": {
            "aws:SourceArn": "arn:aws:cloudtrail:<region code>:<account id>:trail/<trail name>"
        }
    }
}
```

[AWS Documentation](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/create-kms-key-policy-for-cloudtrail.html#create-kms-key-policy-for-cloudtrail-encrypt)

## Granting decrypt permissions

> Before you add your KMS key to your CloudTrail configuration, it is important to give decrypt permissions to all users who require them.
> Users who have encrypt permissions but no decrypt permissions will not be able to read encrypted logs.
> If you are using an existing S3 bucket with an S3 Bucket Key, `kms:Decrypt` permissions are required to create or update a trail with SSE-KMS encryption enabled.

``` json linenums="1" hl_lines="5"
{
    "Sid": "Enable CloudTrail log decrypt permissions",
    "Effect": "Allow",
    "Principal": {
        "AWS": "arn:aws:iam::<account id>:user/<user name>"
    },
    "Action": "kms:Decrypt",
    "Resource": "*",
    "Condition": {
        "Null": {
            "kms:EncryptionContext:aws:cloudtrail:arn": "false"
        }
    }
}
```

[AWS Documentation](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/create-kms-key-policy-for-cloudtrail.html#create-kms-key-policy-for-cloudtrail-decrypt)

## Enable CloudTrail to describe KMS key properties

``` json linenums="1" hl_lines="8 11"
{
    "Sid": "Allow CloudTrail access",
    "Effect": "Allow",
    "Principal": {
        "Service": "cloudtrail.amazonaws.com"
    },
    "Action": "kms:DescribeKey",
    "Resource": "arn:aws:kms:<region code>:<account id>:key/<key id>",
    "Condition": {
        "StringEquals": {
            "aws:SourceArn": "arn:aws:cloudtrail:<region code>:<account id>:trail/<trail name>"
        }
    }
}
```

[AWS Documentation](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/create-kms-key-policy-for-cloudtrail.html#create-kms-key-policy-for-cloudtrail-decrypt)