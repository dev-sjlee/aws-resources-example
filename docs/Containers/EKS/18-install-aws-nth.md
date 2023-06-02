---
title: Install AWS Node Termination Handler (NTH)
Description: Install AWS Node Terminatio Handler (NTH) to use spot instance In EKS cluster.
---

# Install AWS Node Termination Handler (NTH)

## Setup Infrastructure

### Create an SQS Queue and EventBridge Rules

=== ":simple-linux: Linux"

    ``` bash hl_lines="1 2 3 6 7 10 13"
    STACK_NAME="<stack name>"
    PROJECT_NAME="<project name>"
    REGION="<region code>"

    ### SQS Configuration
    QueueName=""            # [optional] The name of SQS queue.
    KmsKeyId=""             # [optional] The KMS key id for SQS queue encryption.

    ### EventBridge Configuration
    EventRulesBaseName=""   # [optional] The base name of EvnetBridge event rules.

    ### IAM Policy Configuration
    IamPolicyName=""        # [REQUIRED] The name of service accounts"s IAM policy.

    curl -LO https://raw.githubusercontent.com/marcus16-kang/aws-resources-example/main/scripts/eks/nth-cfn.yaml

    aws cloudformation deploy \
        --template-file ./nth-cfn.yaml \
        --stack-name $STACK_NAME \
        --parameter-overrides \
            ProjectName=$PROJECT_NAME \
            QueueName=$QueueName \
            KmsKeyId=$KmsKeyId \
            EventRulesBaseName=$EventRulesBaseName \
            IamPolicyName=$IamPolicyName \
        --tags project=$PROJECT_NAME \
        --capabilities CAPABILITY_NAMED_IAM \
        --disable-rollback \
        --region $REGION
    ```

=== ":simple-windows: Windows"

    ``` powershell hl_lines="1 2 3 6 7 10 13"
    $STACK_NAME="<stack name>"
    $PROJECT_NAME="<project name>"
    $REGION="<region code>"

    ### SQS Configuration
    $QueueName=""           # [optional] The name of SQS queue.
    $KmsKeyId=""            # [optional] The KMS key id for SQS queue encryption.

    ### EventBridge Configuration
    $EventRulesBaseName=""  # [optional] The base name of EvnetBridge event rules.

    ### IAM Policy Configuration
    $IamPolicyName=""       # [REQUIRED] The name of service accounts"s IAM policy.

    curl.exe -LO https://raw.githubusercontent.com/marcus16-kang/aws-resources-example/main/scripts/eks/nth-cfn.yaml

    aws cloudformation deploy `
        --template-file ./nth-cfn.yaml `
        --stack-name $STACK_NAME `
        --parameter-overrides `
            ProjectName=$PROJECT_NAME `
            QueueName=$QueueName `
            KmsKeyId=$KmsKeyId `
            EventRulesBaseName=$EventRulesBaseName `
            IamPolicyName=$IamPolicyName `
        --tags project=$PROJECT_NAME `
        --capabilities CAPABILITY_NAMED_IAM `
        --disable-rollback `
        --region $REGION
    ```

!!! note

    If you use CMK, you should add below policy.

    ``` json title="KMS Key Policy"
    {
        "Sid": "Allow EventBridge to use the key",
        "Effect": "Allow",
        "Principal": {
            "Service": "events.amazonaws.com"
        },
        "Action": [
            "kms:Decrypt",
            "kms:GenerateDataKey"
        ],
        "Resource": "*"
    }
    ```

### Create an ASG Termination Lifecycle Hook

=== ":simple-linux: Linux"

    ``` bash hl_lines="1 2 3"
    AUTO_SCALING_GROUP_NAME="<ec2 auto scaling group name>"
    LIFECYCLE_HOOK_NAME="<lifecycle hook name (ex. k8s-term-hook)>"
    REGION="<region code>"

    aws autoscaling put-lifecycle-hook \
        --lifecycle-hook-name=$LIFECYCLE_HOOK_NAME \
        --auto-scaling-group-name=$AUTO_SCALING_GROUP_NAME \
        --lifecycle-transition=autoscaling:EC2_INSTANCE_TERMINATING \
        --default-result=CONTINUE \
        --heartbeat-timeout=300 \
        --region $REGION
    ```

=== ":simple-windows: Windows"

    ``` powershell hl_lines="1 2 3"
    $AUTO_SCALING_GROUP_NAME="<ec2 auto scaling group name>"
    $LIFECYCLE_HOOK_NAME="<lifecycle hook name (ex. k8s-term-hook)>"
    $REGION="<region code>"

    aws autoscaling put-lifecycle-hook `
        --lifecycle-hook-name=$LIFECYCLE_HOOK_NAME `
        --auto-scaling-group-name=$AUTO_SCALING_GROUP_NAME `
        --lifecycle-transition=autoscaling:EC2_INSTANCE_TERMINATING `
        --default-result=CONTINUE `
        --heartbeat-timeout=300 `
        --region $REGION
    ```

### Tag the Instances

=== ":simple-linux: Linux"

    ``` bash hl_lines="1 2"
    AUTO_SCALING_GROUP_NAME="<ec2 auto scaling group name>"
    REGION="<region code>"

    aws autoscaling create-or-update-tags \
        --tags ResourceId=$AUTO_SCALING_GROUP_NAME,ResourceType=auto-scaling-group,Key=aws-node-termination-handler/managed,Value=,PropagateAtLaunch=true \
        --region $REGION

    for i in $(aws autoscaling describe-auto-scaling-groups \
        --auto-scaling-group-name $AUTO_SCALING_GROUP_NAME \
        --region $REGION \
        --query "AutoScalingGroups[0].Instances[*].InstanceId" \
    | jq -r ".[]")
    do
        aws ec2 create-tags \
            --resources $i \
            --tags 'Key="aws-node-termination-handler/managed",Value=' \
            --region $REGION
    done
    ```

=== ":simple-windows: Windows"

    ``` powershell hl_lines="1 2"
    $AUTO_SCALING_GROUP_NAME="<ec2 auto scaling group name>"
    $REGION="<region code>"

    aws autoscaling create-or-update-tags `
        --tags ResourceId=$AUTO_SCALING_GROUP_NAME,ResourceType=auto-scaling-group,Key=aws-node-termination-handler/managed,Value=,PropagateAtLaunch=true `
        --region $REGION

    $InstanceIds = aws autoscaling describe-auto-scaling-groups `
        --auto-scaling-group-name $AUTO_SCALING_GROUP_NAME `
        --region $REGION `
        --query "AutoScalingGroups[0].Instances[*].InstanceId" |
        ConvertFrom-Json

    foreach ($InstanceId in $InstanceIds)
    {
        aws ec2 create-tags `
            --resources $InstanceId `
            --tags 'Key="aws-node-termination-handler/managed",Value=' `
            --region $REGION
    }
    ```

## Install AWS Node Termination Handler using `helm`

### Create ServiceAccount using IAM Role

=== ":simple-linux: Linux"

    ``` bash hl_lines="1 2 3 4 5"
    CLUSTER_NAME="<cluster name>"
    POLICY_NAME="<policy name>"
    ROLE_NAME="<role name>"
    PROJECT_NAME="<project name>"
    REGION="<region code>"

    POLICY_ARN=$(aws iam list-policies \
        --query "Policies[?PolicyName=='$POLICY_NAME'].Arn" --output text)

    eksctl create iamserviceaccount \
        --cluster $CLUSTER_NAME \
        --name aws-node-termination-handler \
        --namespace kube-system \
        --attach-policy-arn $POLICY_ARN \
        --role-name $ROLE_NAME \
        --tags project=$PROJECT_NAME \
        --region $REGION \
        --approve
    ```

=== ":simple-windows: Windows"

    ``` powershell hl_lines="1 2 3 4 5"
    $CLUSTER_NAME="<cluster name>"
    $POLICY_NAME="<policy name>"
    $ROLE_NAME="<role name>"
    $PROJECT_NAME="<project name>"
    $REGION="<region code>"

    $POLICY_ARN = aws iam list-policies `
        --query "Policies[?PolicyName=='$POLICY_NAME'].Arn" --output text

    eksctl create iamserviceaccount `
        --cluster $CLUSTER_NAME `
        --name aws-node-termination-handler `
        --namespace kube-system `
        --attach-policy-arn $POLICY_ARN `
        --role-name $ROLE_NAME `
        --tags project=$PROJECT_NAME `
        --region $REGION `
        --approve
    ```

!!! note

    If you use CMK, you should add below policy.

    ``` json title="KMS Key Policy" hl_lines="6"
    {
        "Sid": "AllowConsumersToReceiveFromTheQueue - NTH",
        "Effect": "Allow",
        "Principal": {
            "AWS": [
                "<NTH SERVICE ACCOUNT ROLE ARN>"
            ]
        },
        "Action": [
            "kms:Decrypt"
        ],
        "Resource": "*"
    }
    ```


[Github](https://github.com/aws/aws-node-termination-handler)

### Deploy using `helm`

=== ":simple-linux: Linux"

    ``` bash hl_lines="1 2"
    STACK_NAME="<stack name>"   # NTH infra stack name.
    REGION="<region code>"

    QUEUE_URL=$(aws cloudformation describe-stacks \
        --stack-name $STACK_NAME \
        --query "Stacks[0].Outputs[?OutputKey=='QueueURL'].OutputValue" \
        --region $REGION \
        --output text)

    helm repo add eks https://aws.github.io/eks-charts

    helm install aws-node-termination-handler eks/aws-node-termination-handler \
        --namespace kube-system \
        --set serviceAccount.create=false \
        --set serviceAccount.name=aws-node-termination-handler \
        --set enableSqsTerminationDraining=true \
        --set queueURL=$QUEUE_URL \
        --set awsRegion=$REGION
    ```

=== ":simple-windows: Windows"

    ``` powershell hl_lines="1 2"
    $STACK_NAME="<stack name>"   # NTH infra stack name.
    $REGION="<region code>"

    $QUEUE_URL = aws cloudformation describe-stacks `
        --stack-name $STACK_NAME `
        --query "Stacks[0].Outputs[?OutputKey=='QueueURL'].OutputValue" `
        --region $REGION `
        --output text

    helm repo add eks https://aws.github.io/eks-charts

    helm install aws-node-termination-handler eks/aws-node-termination-handler `
        --namespace kube-system `
        --set serviceAccount.create=false `
        --set serviceAccount.name=aws-node-termination-handler `
        --set enableSqsTerminationDraining=true `
        --set queueURL=$QUEUE_URL `
        --set awsRegion=$REGION
    ```

[Helm Chart](https://github.com/aws/eks-charts/tree/master/stable/aws-node-termination-handler)