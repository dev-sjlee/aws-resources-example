---
title: Create a S3 Bucket
description: Create a S3 bucket using CloudFormation.
---

# Create a S3 Bucket

=== ":simple-linux: Linux"

    ``` bash
    STACK_NAME="<stack name>"
    PROJECT_NAME="<project name>"
    REGION="<region code>"

    # Bucket Configuration - General
    BucketName=""                           # [REQUIRED] The name of S3 bucket.

    # Bucket Configuration - Encryption
    BucketEncryptionAlgorithm="aws:kms"     # `aws:kms`(default) or `AES256` | [REQUIRED] The type of S3 bucket server-side encryption.
    BucketEncryptionKmsKeyId=""             # [optional] The KMS key id or arn for S3 bucket server-side encryption.

    # Bucket Configuration - Logging
    LoggingDestinationBucketName=""         # [optional] The log prefix of S3 bucket logging.
    LoggingDestinationPrefix=""             # [optional] The log prefix of S3 bucket logging.

    curl -LO https://raw.githubusercontent.com/marcus16-kang/cloudformation-templates/main/s3/bucket.yaml

    aws cloudformation deploy \
        --template-file ./bucket.yaml \
        --stack-name $STACK_NAME \
        --parameter-overrides \
            ProjectName=$ProjectName \
            BucketName=$BucketName \
            BucketEncryptionAlgorithm=$BucketEncryptionAlgorithm \
            BucketEncryptionKmsKeyId=$BucketEncryptionKmsKeyId \
            LoggingDestinationBucketName=$LoggingDestinationBucketName \
            LoggingDestinationPrefix=$LoggingDestinationPrefix \
        --disable-rollback \
        --tags project=$PROJECT_NAME \
        --region $REGION
    ```

=== ":simple-windows: Windows"

    ``` powershell
    $STACK_NAME="<stack name>"
    $PROJECT_NAME="<project name>"
    $REGION="<region code>"

    # Bucket Configuration - General
    $BucketName=""                          # [REQUIRED] The name of S3 bucket.

    # Bucket Configuration - Encryption
    $BucketEncryptionAlgorithm="aws:kms"    # `aws:kms`(default) or `AES256` | [REQUIRED] The type of S3 bucket server-side encryption.
    $BucketEncryptionKmsKeyId=""            # [optional] The KMS key id or arn for S3 bucket server-side encryption.

    # Bucket Configuration - Logging
    $LoggingDestinationBucketName=""        # [optional] The log prefix of S3 bucket logging.
    $LoggingDestinationPrefix=""            # [optional] The log prefix of S3 bucket logging.

    curl.exe -LO https://raw.githubusercontent.com/marcus16-kang/cloudformation-templates/main/s3/bucket.yaml

    aws cloudformation deploy `
        --template-file ./bucket.yaml `
        --stack-name $STACK_NAME `
        --parameter-overrides `
            ProjectName=$ProjectName `
            BucketName=$BucketName `
            BucketEncryptionAlgorithm=$BucketEncryptionAlgorithm `
            BucketEncryptionKmsKeyId=$BucketEncryptionKmsKeyId `
            LoggingDestinationBucketName=$LoggingDestinationBucketName `
            LoggingDestinationPrefix=$LoggingDestinationPrefix `
        --disable-rollback `
        --tags project=$PROJECT_NAME `
        --region $REGION
    ```