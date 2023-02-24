---
description: Enable encrypt and decrypt using CMK to CloudFront distributions.
---

# CloudFront KMS Policies

## Granting encrypt and decrypt permissions to CloudFront.

``` json linenums="1" hl_lines="17"
{
    "Sid": "Allow users or roles to use KMS to CloudFront.",
    "Effect": "Allow",
    "Principal": {
        "Service": [
            "cloudfront.amazonaws.com"
        ]
     },
    "Action": [
        "kms:Decrypt",
        "kms:Encrypt",
        "kms:GenerateDataKey*"
    ],
    "Resource": "*",
    "Condition":{
        "StringEquals":{
            "aws:SourceArn": "arn:aws:cloudfront::<account id>:distribution/<cloudfront distribution id>"
        }
    }
}
```

[AWS Documentation](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/private-content-restricting-access-to-s3.html)