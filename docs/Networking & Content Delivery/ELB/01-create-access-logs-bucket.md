---
titie: Create a bucket for saving access logs.
description: Create a bucket for saving access logs.
---

# Create a bucket for saving access logs

=== "Linux (Bash) :simple-linux:"

    ``` bash
    STACK_NAME="<stack name>"
    PROJECT_NAME="<project name>"
    REGION="<region code>"

    ### Bucket Configuration
    AccessLogBucketName=""  # [REQUIRED] The name of alb access log bucket.
    AccessLogPrefix=""      # [optional] The prefix of alb access log. It cannot start or end with `/`.

    curl -LO https://raw.githubusercontent.com/marcus16-kang/cloudformation-templates/main/elb/access-log-bucket.yaml

    aws cloudformation deploy \
        --template-file ./access-log-bucket.yaml \
        --stack-name $STACK_NAME \
        --parameter-overrides \
            ProjectName=$PROJECT_NAME \
            AccessLogBucketName=$AccessLogBucketName \
            AccessLogPrefix=$AccessLogPrefix \
        --disable-rollback \
        --tags project=$PROJECT_NAME \
        --region $REGION
    ```

=== "Windows (Powershell) :simple-windows:"

    ``` powershell
    $STACK_NAME="<stack name>"
    $PROJECT_NAME="<project name>"
    $REGION="<region code>"

    ### Bucket Configuration
    $AccessLogBucketName="" # [REQUIRED] The name of alb access log bucket.
    $AccessLogPrefix=""     # [optional] The prefix of alb access log. It cannot start or end with `/`.

    curl.exe -LO https://raw.githubusercontent.com/marcus16-kang/cloudformation-templates/main/elb/access-log-bucket.yaml

    aws cloudformation deploy `
        --template-file ./access-log-bucket.yaml `
        --stack-name $STACK_NAME `
        --parameter-overrides `
            ProjectName=$PROJECT_NAME `
            AccessLogBucketName=$AccessLogBucketName `
            AccessLogPrefix=$AccessLogPrefix `
        --disable-rollback `
        --tags project=$PROJECT_NAME `
        --region $REGION
    ```