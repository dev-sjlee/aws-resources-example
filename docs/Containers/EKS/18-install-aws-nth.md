---
title: Install AWS Node Termination Handler (NTH)
Description: Install AWS Node Terminatio Handler (NTH) to use spot instance In EKS cluster.
---

# Install AWS Node Termination Handler (NTH)

## Setup Infrastructure

### Create an SQS Queue and EventBridge Rules

``` shell hl_lines="1 2 3 4 5 6"
NTH_INFRA_STACK_NAME="<stack name>"
# NTH_QUEUE_NAME="<queue name>"
# NTH_QUEUE_KMS_ID="<kms key id>"
# NTH_EVENT_RULES_BASE_NAME="<event rules base name>"
PROJECT_NAME="<project name>"
REGION="<region code>"

curl -LO https://raw.githubusercontent.com/marcus16-kang/aws-resources-example/main/scripts/eks/nth-cfn.yaml

aws cloudformation deploy \
    --template-file ./nth-cfn.yaml \
    --stack-name $NTH_INFRA_STACK_NAME \
    --parameter-overrides \
        ProjectName=$PROJECT_NAME \
        # QueueName=$NTH_QUEUE_NAME \
        # KmsKeyId=$NTH_QUEUE_KMS_ID \
        # EventRulesBaseName=$NTH_EVENT_RULES_BASE_NAME \
    --tags project=$PROJECT_NAME \
    --region $REGION
```

### Create an ASG Termination Lifecycle Hook

``` shell hl_lines="1 2 3"
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

### Tag the Instances
``` shell hl_lines="1 2"
AUTO_SCALING_GROUP_NAME="<ec2 auto scaling group name>"
REGION="<region code>"

aws autoscaling create-or-update-tags \
    --tags ResourceId=$AUTO_SCALING_GROUP_NAME,ResourceType=auto-scaling-group,Key=aws-node-termination-handler/managed,Value=,PropagateAtLaunch=true

for i in $(aws autoscaling describe-auto-scaling-groups \
    --auto-scaling-group-name $AUTO_SCALING_GROUP_NAME \
    --region $REGION \
    --query "AutoScalingGroups[0].Instances[*].InstanceId" \
| jq -r ".[]")
do
    aws ec2 create-tags \
        --resources $i \
        --tags 'Key="aws-node-termination-handler/managed",Value='
done
```

## Install AWS Node Termination Handler using `helm`

### Create an IAM policy for ServiceAccount

``` shell hl_lines="1 2"
POLICY_NAME="<policy name>"
PROJECT_NAME="<project name>"

cat << EOF > nth-policy.json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "autoscaling:CompleteLifecycleAction",
                "autoscaling:DescribeAutoScalingInstances",
                "autoscaling:DescribeTags",
                "ec2:DescribeInstances",
                "sqs:DeleteMessage",
                "sqs:ReceiveMessage"
            ],
            "Resource": "*"
        }
    ]
}
EOF

aws iam create-policy \
    --policy-name $POLICY_NAME \
    --policy-document file://nth-policy.json \
    # --tags Key=project,Value=$PROJECT_NAME
```

### Deploy using `helm`

``` shell hl_lines="1 2"
NTH_INFRA_STACK_NAME="<stack name>"
REGION="<region code>"

QUEUE_URL=$(aws cloudformation describe-stacks \
    --stack-name $NTH_INFRA_STACK_NAME \
    --query "Stacks[0].Outputs[?OutputKey=='QueueURL'].OutputValue" \
    --output text)

helm repo add eks https://aws.github.io/eks-charts

helm upgrade --install aws-node-termination-handler \
    --namespace kube-system \
    --set enableSqsTerminationDraining=true \
    --set queueURL=$QUEUE_URL \
    --set awsRegion=$REGION \
    eks/aws-node-termination-handler
```

[Helm Chart](https://github.com/aws/eks-charts/tree/master/stable/aws-node-termination-handler)

### Create ServiceAccount using IAM Role

``` shell hl_lines="1 2 3 4 5"
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
    --override-existing-serviceaccounts \
    --approve

kubectl rollout restart deployment/aws-node-termination-handler -n kube-system
```

[Github](https://github.com/aws/aws-node-termination-handler)