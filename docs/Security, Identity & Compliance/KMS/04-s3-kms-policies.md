---
description: Enable encrypt and decrypt using CMK to S3 buckets.
---

# S3 KMS Policies

## Granting encrypt and decrypt permissions to users or roles.

``` json linenums="1" hl_lines="8 12 16"
{
    "Sid": "Allow users or roles to use KMS to S3.",
    "Effect": "Allow",
    "Principal": {
        "AWS": "arn:aws:iam::<account id>:<users or roles>/<users or roles name>"
    },
    "Action": [
        "kms:Encrypt",
        "kms:Decrypt",
        "kms:ReEncrypt*",
        "kms:GenerateDataKey*",
        "kms:DescribeKey"
    ],
    "Resource": "arn:aws:kms:<region code>:<account id>:key/<kms key id>"
}
```

## Requiring server-side encryption

``` json linenums="1"
{
   "Version":"2012-10-17",
   "Id":"PutObjectPolicy",
   "Statement":[{
         "Sid":"DenyUnEncryptedObjectUploads",
         "Effect":"Deny",
         "Principal":"*",
         "Action":"s3:PutObject",
         "Resource":"arn:aws:s3:::<bucket name>/*",
         "Condition":{
            "StringNotEquals":{
               "s3:x-amz-server-side-encryption":"aws:kms"
            }
         }
      }
   ]
}
```

If you want to upload using sse-kms, please see [this documentation](https://docs.aws.amazon.com/AmazonS3/latest/userguide/specifying-s3-encryption.html).

``` shell
aws s3 cp ./2022-11-03_14:11:49 s3://samsung-kms-test --sse aws:kms
```

[AWS Documentation](https://docs.aws.amazon.com/AmazonS3/latest/userguide/UsingKMSEncryption.html)

## S3 Replication Policies

### KMS Policies

``` json linenums="1"
{
   "Sid": "AllowS3ReplicationSourceRoleToUseTheKey",
   "Effect": "Allow",
   "Principal": {
         "AWS": "arn:aws:iam::<account id>:role/service-role/<s3 replication role name>"
   },
   "Action": [
         "kms:GenerateDataKey",
         "kms:Encrypt",
         "kms:Decrypt"
   ],
   "Resource": "*"
},
{
   "Sid": "AllowS3ReplicationDestinationRoleToUseTheKey",
   "Effect": "Allow",
   "Principal": {
         "AWS": "arn:aws:iam::<account id>:role/service-role/<s3 replication role name>"
   },
   "Action": [
         "kms:GenerateDataKey",
         "kms:Encrypt",
         "kms:Decrypt"
   ],
   "Resource": "*"
}
```

!!! note
    You should update policies both key(source and destination).