---
title: Create ECS Service
description: Create ECS service using CloudFormation.
---

# Create ECS Service

=== ":simple-linux: Linux"

    ``` bash
    STACK_NAME="<stack name>"
    PROJECT_NAME="<project name>"
    REGION="<region code>"

    ### Service Configuration - General
    ClusterName=""              # [REQUIRED] The name of ECS cluster.
    ServiceName=""              # [REQUIRED] The name of this service.
    CapacityProviderName=""     # `FARGATE` or capacity provider name | [REQUIRED] The name of capacity provider.
    DesiredCount=""             # [REQUIRED] The number of task in service.
    TaskDefinition=""           # [REQUIRED] The family and revision (family:revision).
    DeploymentType="ECS"        # `ECS`(default) or `CODE_DEPLOY` | [REQUIRED] The type of service's deployment type.

    ### Service Configuration - Load Balancer
    ContainerName=""            # [REQUIRED] The name of the container to associate with the laod balancer.
    ContainerPort=""            # [REQUIRED] The port on the container to associate with the load balancer.
    TargetGroupArn=""           # [REQUIRED] The ARN of the ALB target group.

    ### Service Configuration - Network
    SecurityGroups=""           # [REQUIRED] The IDs of the security groups associated with the service.
    Subnets=""                  # [REQUIRED] The IDs of the subents associated with the service.

    ### Service Configuration - Service Discovery
    CloudMapServiceId=""        # [optional] The ID of Cloud Map service for service discovery in ECS.

    curl -LO https://raw.githubusercontent.com/marcus16-kang/cloudformation-templates/main/ecs/service/service.yaml

    aws cloudformation deploy \
        --stack-name $STACK_NAME \
        --template-file ./service.yaml \
        --parameter-overrides \
            ProjectName=$PROJECT_NAME \
            ClusterName=$ClusterName \
            ServiceName=$ServiceName \
            CapacityProviderName=$CapacityProviderName \
            DesiredCount=$DesiredCount \
            TaskDefinition=$TaskDefinition \
            DeploymentType=$DeploymentType \
            ContainerName=$ContainerName \
            ContainerPort=$ContainerPort \
            TargetGroupArn=$TargetGroupArn \
            SecurityGroups=$SecurityGroups \
            Subnets=$Subnets \
            CloudMapServiceId=$CloudMapServiceId \
        --tags project=$PROJECT_NAME \
        --capabilities CAPABILITY_NAMED_IAM \
        --disable-rollback \
        --region $REGION
    ```

=== ":simple-windows: Windows"

    ``` powershell
    $STACK_NAME="<stack name>"
    $PROJECT_NAME="<project name>"
    $REGION="<region code>"

    ### Service Configuration - General
    $ClusterName=""             # [REQUIRED] The name of ECS cluster.
    $ServiceName=""             # [REQUIRED] The name of this service.
    $CapacityProviderName=""    # `FARGATE` or capacity provider name | [REQUIRED] The name of capacity provider.
    $DesiredCount=""            # [REQUIRED] The number of task in service.
    $TaskDefinition=""          # [REQUIRED] The family and revision (family:revision).
    $DeploymentType="ECS"       # `ECS`(default) or `CODE_DEPLOY` | [REQUIRED] The type of service's deployment type.

    ### Service Configuration - Load Balancer
    $ContainerName=""           # [REQUIRED] The name of the container to associate with the laod balancer.
    $ContainerPort=""           # [REQUIRED] The port on the container to associate with the load balancer.
    $TargetGroupArn=""          # [REQUIRED] The ARN of the ALB target group.

    ### Service Configuration - Network
    $SecurityGroups=""          # [REQUIRED] The IDs of the security groups associated with the service.
    $Subnets=""                 # [REQUIRED] The IDs of the subents associated with the service.

    ### Service Configuration - Service Discovery
    $CloudMapServiceId=""       # [optional] The ID of Cloud Map service for service discovery in ECS.

    curl.exe -LO https://raw.githubusercontent.com/marcus16-kang/cloudformation-templates/main/ecs/service/service.yaml

    aws cloudformation deploy `
        --stack-name $STACK_NAME `
        --template-file ./service.yaml `
        --parameter-overrides `
            ProjectName=$PROJECT_NAME `
            ClusterName=$ClusterName `
            ServiceName=$ServiceName `
            CapacityProviderName=$CapacityProviderName `
            DesiredCount=$DesiredCount `
            TaskDefinition=$TaskDefinition `
            DeploymentType=$DeploymentType `
            ContainerName=$ContainerName `
            ContainerPort=$ContainerPort `
            TargetGroupArn=$TargetGroupArn `
            SecurityGroups=$SecurityGroups `
            Subnets=$Subnets `
            CloudMapServiceId=$CloudMapServiceId `
        --tags project=$PROJECT_NAME `
        --capabilities CAPABILITY_NAMED_IAM `
        --disable-rollback `
        --region $REGION
    ```