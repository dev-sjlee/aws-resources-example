---
title: Create ECS Cluster
description: Create ECS cluster using CloudFormation.
---

# Create ECS Cluster

=== ":simple-linux: Linux"

    ``` bash
    STACK_NAME="<stack name>"
    PROJECT_NAME="<project name>"
    REGION="<region code>"

    ### ECS Cluster Configuration - General
    ClusterName=""                              # [REQUIRED] The name of ECS cluster.

    ### ECS Cluster Configuration - Container Insights
    ContainerInsights="enabled"                 # `enabled`(default) of `disabled` | [optional] Enable of disable ECS cluster's container insights.

    ### ECS Cluster Configuration - Alarm
    EnableClusterAlarms="true"                  # `true`(default) of `false` | [optional] Enable of disable to create ECS cluster's alarms.
    CpuUtilizationAlarmName=""                  # [optional] The name of CpuUtilizationAlarm.
    MemoryUtilizationAlarmName=""               # [optional] The name of MemoryUtilizationAlarm.
    CpuUtilizationThreshold="70"                # [optional] The number of CPUUtilization threshold. Default is 70(%).
    MemoryUtilizationThreshold="70"             # [optional] The number of MemoryUtilization threshold. Default is 70(%).

    ### ECS Capacity Provider Configuration - EC2
    EnableEc2CapacityProvider="false"           # `true`(default) of `false` | [REQUIRED] Enable or disable EC2 capacity provider for ECS cluster.
    Ec2VpcId=""                                 # [REQUIRED] The VPC id of EC2 capacity provider for ECS cluster.
    Ec2securityGroupName=""                     # [REQUIRED] The security group name of EC2 capacity provider for ECS cluster.
    BastionSecurityGroupForEc2=""               # [optional] The security group id of bastion security group.
    Ec2RoleName=""                              # [REQUIRED] The name of EC2 capacity provider for ECS cluster.
    EC2InstanceName=""                          # [REQUIRED] The instance name of EC2 capacity provider for ECS cluster.
    Ec2LaunchTemplateName=""                    # [optional] The launch template name of EC2 capacity provider for ECS cluster.
    Ec2InstanceArchitecture=""                  # `x86-64` of `arm64` | [REQUIRED] The instance architecture of EC2 capacity provider for ECS cluster.
    Ec2InstanceOs=""                            # `amazonlinux2` of `bottlerocket` | [REQUIRED] The instance os of EC2 capacity provider for ECS cluster.
    Ec2InstanceType=""                          # [REQUIRED] The instance type of EC2 capacity provider for ECS cluster.
    Ec2KeyName=""                               # [optional] The key pair name of EC2 capacity provider for ECS cluster.
    Ec2SshPort="22"                             # [optional] The SSH port number of EC2 capacity provider for ECS cluster.
    Ec2AutoScalingGroupName=""                  # [optional] The auto scaling group name of EC2 capacity provider for ECS cluster.
    Ec2AutoScalingDesiredSize="2"               # [REQUIRED] The desired size of EC2 AutoScaling Group.
    Ec2AutoScalingMinSize="2"                   # [REQUIRED] The min size of EC2 AutoScaling Group.
    Ec2AutoScalingMaxSize="10"                  # [REQUIRED] The max size of EC2 AutoScaling Group.
    Ec2AutoScalingSubnetIds=""                  # [REQUIRED] The subnet id list of EC2 AutoScaling Group.

    ### ECS Capacity Provider Configuration - Fargate
    EnableFargateCapacityProvider="false"       # `false`(default) of `true` | [REQUIRED] Enable or disable Fargate capacity provider for ECS cluster.

    curl -LO https://raw.githubusercontent.com/marcus16-kang/cloudformation-templates/main/ecs/cluster/cluster.yaml

    aws cloudformation deploy \
        --stack-name $STACK_NAME \
        --template-file ./cluster.yaml \
        --parameter-overrides \
            ProjectName=$PROJECT_NAME \
            ClusterName=$ClusterName \
            ContainerInsights=$ContainerInsights \
            EnableClusterAlarms=$EnableClusterAlarms \
            CpuUtilizationAlarmName=$CpuUtilizationAlarmName \
            MemoryUtilizationAlarmName=$MemoryUtilizationAlarmName \
            CpuUtilizationThreshold=$CpuUtilizationThreshold \
            MemoryUtilizationThreshold=$MemoryUtilizationThreshold \
            EnableEc2CapacityProvider=$EnableEc2CapacityProvider \
            Ec2VpcId=$Ec2VpcId \
            Ec2securityGroupName=$Ec2securityGroupName \
            BastionSecurityGroupForEc2=$BastionSecurityGroupForEc2 \
            Ec2RoleName=$Ec2RoleName \
            EC2InstanceName=$EC2InstanceName \
            Ec2LaunchTemplateName=$Ec2LaunchTemplateName \
            Ec2InstanceArchitecture=$Ec2InstanceArchitecture \
            Ec2InstanceOs=$Ec2InstanceOs \
            Ec2InstanceType=$Ec2InstanceType \
            Ec2KeyName=$Ec2KeyName \
            Ec2SshPort=$Ec2SshPort \
            Ec2AutoScalingGroupName=$Ec2AutoScalingGroupName \
            Ec2AutoScalingDesiredSize=$Ec2AutoScalingDesiredSize \
            Ec2AutoScalingMinSize=$Ec2AutoScalingMinSize \
            Ec2AutoScalingMaxSize=$Ec2AutoScalingMaxSize \
            Ec2AutoScalingSubnetIds=$Ec2AutoScalingSubnetIds \
            EnableFargateCapacityProvider=$EnableFargateCapacityProvider \
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

    ### ECS Cluster Configuration - General
    $ClusterName=""                             # [REQUIRED] The name of ECS cluster.

    ### ECS Cluster Configuration - Container Insights
    $ContainerInsights="enabled"                # `enabled`(default) of `disabled` | [optional] Enable of disable ECS cluster's container insights.

    ### ECS Cluster Configuration - Alarm
    $EnableClusterAlarms="true"                 # `true`(default) of `false` | [optional] Enable of disable to create ECS cluster's alarms.
    $CpuUtilizationAlarmName=""                 # [optional] The name of CpuUtilizationAlarm.
    $MemoryUtilizationAlarmName=""              # [optional] The name of MemoryUtilizationAlarm.
    $CpuUtilizationThreshold="70"               # [optional] The number of CPUUtilization threshold. Default is 70(%).
    $MemoryUtilizationThreshold="70"            # [optional] The number of MemoryUtilization threshold. Default is 70(%).

    ### ECS Capacity Provider Configuration - EC2
    $EnableEc2CapacityProvider="false"          # `true`(default) of `false` | [REQUIRED] Enable or disable EC2 capacity provider for ECS cluster.
    $Ec2VpcId=""                                # [REQUIRED] The VPC id of EC2 capacity provider for ECS cluster.
    $Ec2securityGroupName=""                    # [REQUIRED] The security group name of EC2 capacity provider for ECS cluster.
    $BastionSecurityGroupForEc2=""              # [optional] The security group id of bastion security group.
    $Ec2RoleName=""                             # [REQUIRED] The name of EC2 capacity provider for ECS cluster.
    $EC2InstanceName=""                         # [REQUIRED] The instance name of EC2 capacity provider for ECS cluster.
    $Ec2LaunchTemplateName=""                   # [optional] The launch template name of EC2 capacity provider for ECS cluster.
    $Ec2InstanceArchitecture=""                 # `x86-64` of `arm64` | [REQUIRED] The instance architecture of EC2 capacity provider for ECS cluster.
    $Ec2InstanceOs=""                           # `amazonlinux2` of `bottlerocket` | [REQUIRED] The instance os of EC2 capacity provider for ECS cluster.
    $Ec2InstanceType=""                         # [REQUIRED] The instance type of EC2 capacity provider for ECS cluster.
    $Ec2KeyName=""                              # [optional] The key pair name of EC2 capacity provider for ECS cluster.
    $Ec2SshPort="22"                            # [optional] The SSH port number of EC2 capacity provider for ECS cluster.
    $Ec2AutoScalingGroupName=""                 # [optional] The auto scaling group name of EC2 capacity provider for ECS cluster.
    $Ec2AutoScalingDesiredSize="2"              # [REQUIRED] The desired size of EC2 AutoScaling Group.
    $Ec2AutoScalingMinSize="2"                  # [REQUIRED] The min size of EC2 AutoScaling Group.
    $Ec2AutoScalingMaxSize="10"                 # [REQUIRED] The max size of EC2 AutoScaling Group.
    $Ec2AutoScalingSubnetIds=""                 # [REQUIRED] The subnet id list of EC2 AutoScaling Group.

    ### ECS Capacity Provider Configuration - Fargate
    $EnableFargateCapacityProvider="false"      # `false`(default) of `true` | [REQUIRED] Enable or disable Fargate capacity provider for ECS cluster.


    curl.exe -LO https://raw.githubusercontent.com/marcus16-kang/cloudformation-templates/main/ecs/cluster/cluster.yaml

    aws cloudformation deploy `
        --stack-name $STACK_NAME `
        --template-file ./cluster.yaml `
        --parameter-overrides `
            ProjectName=$PROJECT_NAME `
            ClusterName=$ClusterName `
            ContainerInsights=$ContainerInsights `
            EnableClusterAlarms=$EnableClusterAlarms `
            CpuUtilizationAlarmName=$CpuUtilizationAlarmName `
            MemoryUtilizationAlarmName=$MemoryUtilizationAlarmName `
            CpuUtilizationThreshold=$CpuUtilizationThreshold `
            MemoryUtilizationThreshold=$MemoryUtilizationThreshold `
            EnableEc2CapacityProvider=$EnableEc2CapacityProvider `
            Ec2VpcId=$Ec2VpcId `
            Ec2securityGroupName=$Ec2securityGroupName `
            BastionSecurityGroupForEc2=$BastionSecurityGroupForEc2 `
            Ec2RoleName=$Ec2RoleName `
            EC2InstanceName=$EC2InstanceName `
            Ec2LaunchTemplateName=$Ec2LaunchTemplateName `
            Ec2InstanceArchitecture=$Ec2InstanceArchitecture `
            Ec2InstanceOs=$Ec2InstanceOs `
            Ec2InstanceType=$Ec2InstanceType `
            Ec2KeyName=$Ec2KeyName `
            Ec2SshPort=$Ec2SshPort `
            Ec2AutoScalingGroupName=$Ec2AutoScalingGroupName `
            Ec2AutoScalingDesiredSize=$Ec2AutoScalingDesiredSize `
            Ec2AutoScalingMinSize=$Ec2AutoScalingMinSize `
            Ec2AutoScalingMaxSize=$Ec2AutoScalingMaxSize `
            Ec2AutoScalingSubnetIds=$Ec2AutoScalingSubnetIds `
            EnableFargateCapacityProvider=$EnableFargateCapacityProvider `
        --tags project=$PROJECT_NAME `
        --capabilities CAPABILITY_NAMED_IAM `
        --disable-rollback `
        --region $REGION
    ```