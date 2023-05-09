---
title: Create a Trail
description: Create a new trail.
---

# Create a Trail

=== ":simple-linux: Linux"

    ``` bash
    STACK_NAME="<stack name>"
    PROJECT_NAME="<project name>"
    REGION="<region code>"

    ### Trail Configuration - General
    TrailName=""                # [REQUIRED] The name of CloudTrail trail.
    EnableLogGroup="Enable"     # `Enable`(default) or `Disable` | [REQUIRED] Enable of disable logging to CloudWatch logs from CloudTrail trail.
    LogGroupName=""             # [optional] The name of CloudWatch Logs LogGroup for CloudTrail trail.
    LogRoleName=""              # [optional] The name of CloudWatch Logs IAM Role for CloudTrail trail.

    ### Trail Configuration - S3
    BucketName=""               # [optional] The name of bucket.
    BucketPrefix=""             # [optional] The prefix of bucket.

    ### Trail Configuration - KMS
    KmsKeyAlias=""              # [REQUIRED] The name of KMS key. (MUST BE STARTED WITH 'alias/')

    curl -LO https://raw.githubusercontent.com/marcus16-kang/cloudformation-templates/main/cloudtrail/trail.yaml

    aws cloudformation deploy \
        --template-file ./trail.yaml \
        --stack-name $STACK_NAME \
        --parameter-overrides \
            ProjectName=$PROJECT_NAME \
            TrailName=$TrailName \
            EnableLogGroup=$EnableLogGroup \
            LogGroupName=$LogGroupName \
            LogRoleName=$LogRoleName \
            BucketName=$BucketName \
            BucketPrefix=$BucketPrefix \
            KmsKeyAlias=$KmsKeyAlias \
        --capabilities CAPABILITY_NAMED_IAM \
        --disable-rollback \
        --tags project=$PROJECT_NAME \
        --region $REGION
    ```

=== ":simple-windows: Windows"

    ``` powershell
    $STACK_NAME="<stack name>"
    $PROJECT_NAME="<project name>"
    $REGION="<region code>"

    ### Trail Configuration - General
    $TrailName=""               # [REQUIRED] The name of CloudTrail trail.
    $EnableLogGroup="Enable"    # `Enable`(default) or `Disable` | [REQUIRED] Enable of disable logging to CloudWatch logs from CloudTrail trail.
    $LogGroupName=""            # [optional] The name of CloudWatch Logs LogGroup for CloudTrail trail.
    $LogRoleName=""             # [optional] The name of CloudWatch Logs IAM Role for CloudTrail trail.

    ### Trail Configuration - S3
    $BucketName=""              # [optional] The name of bucket.
    $BucketPrefix=""            # [optional] The prefix of bucket.

    ### Trail Configuration - KMS
    $KmsKeyAlias=""             # [REQUIRED] The name of KMS key. (MUST BE STARTED WITH 'alias/')

    curl.exe -LO https://raw.githubusercontent.com/marcus16-kang/cloudformation-templates/main/cloudtrail/trail.yaml

    aws cloudformation deploy `
        --template-file ./trail.yaml `
        --stack-name $STACK_NAME `
        --parameter-overrides `
            ProjectName=$PROJECT_NAME `
            TrailName=$TrailName `
            EnableLogGroup=$EnableLogGroup `
            LogGroupName=$LogGroupName `
            LogRoleName=$LogRoleName `
            BucketName=$BucketName `
            BucketPrefix=$BucketPrefix `
            KmsKeyAlias=$KmsKeyAlias `
        --capabilities CAPABILITY_NAMED_IAM `
        --disable-rollback `
        --tags project=$PROJECT_NAME `
        --region $REGION
    ```