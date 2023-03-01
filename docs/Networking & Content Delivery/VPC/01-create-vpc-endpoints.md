---
title: Create VPC Endpoints
description: Create VPC endpoints using AWS CLI.
---

# Create VPC Endpoints

## Using AWS CLI

### AWS CLI v2

=== "Windows (Powershell) :simple-windows:"

    ``` powershell linenums="1" hl_lines="1 2 3 4 5 8 9"
    $VPC_ID = ""
    $SUBNET_IDS = ("", "")
    $SECURITY_GROUP_ID = ""
    $PROJECT_NAME = ""
    $REGION = ""

    $ENDPOINT_LIST = @(
        ("sts-ep", "sts"),
        ("monitoring-ep", "monitoring")
    )

    foreach ($item in $ENDPOINT_LIST) {
        aws ec2 create-vpc-endpoint `
        --vpc-endpoint-type Interface `
        --vpc-id $VPC_ID `
        --service-name com.amazonaws.$REGION.$($item[1]) `
        --subnet-ids $SUBNET_IDS `
        --security-group-ids $SECURITY_GROUP_ID `
        --ip-address-type ipv4 `
        --tag-specifications "ResourceType=vpc-endpoint,Tags=[{Key=Name,Value=$($item[0])},{Key=project,Value=$PROJECT_NAME}]" `
        --region $REGION `
        --no-cli-pager
    }
    ```

=== "Linux (Bash) :simple-linux:"

    ``` bash linenums="1" hl_lines="1 2 3 4 5 8 9"
    VPC_ID=""
    SUBNET_IDS=""
    SECURITY_GROUP_ID=""
    PROJECT_NAME=""
    REGION=""

    ENDPOINT_LIST=(
        "sts-ep sts"
        "monitoring-ep monitoring"
    )

    for item in "${ENDPOINT_LIST[@]}"
    do
        read -ra list <<< "$item"

        aws ec2 create-vpc-endpoint \
            --vpc-endpoint-type Interface \
            --vpc-id $VPC_ID \
            --service-name com.amazonaws.$REGION.${list[1]} \
            --subnet-ids $SUBNET_IDS \
            --security-group-ids $SECURITY_GROUP_ID \
            --ip-address-type ipv4 \
            --tag-specifications "ResourceType=vpc-endpoint,Tags=[{Key=Name,Value=${list[0]}},{Key=project,Value=$PROJECT_NAME}]" \
            --region $REGION \
            --no-cli-pager
    done
    ```

### AWS CLI v1

=== "Windows (Powershell) :simple-windows:"

    ``` powershell linenums="1" hl_lines="1 2 3 4 5 8 9"
    $VPC_ID = ""
    $SUBNET_IDS = ("", "")
    $SECURITY_GROUP_ID = ""
    $PROJECT_NAME = ""
    $REGION = ""

    $ENDPOINT_LIST = @(
        ("sts-ep", "sts"),
        ("monitoring-ep", "monitoring")
    )

    foreach ($item in $ENDPOINT_LIST) {
        aws ec2 create-vpc-endpoint `
        --vpc-endpoint-type Interface `
        --vpc-id $VPC_ID `
        --service-name com.amazonaws.$REGION.$($item[1]) `
        --subnet-ids $SUBNET_IDS `
        --security-group-ids $SECURITY_GROUP_ID `
        --tag-specifications "ResourceType=vpc-endpoint,Tags=[{Key=Name,Value=$($item[0])},{Key=project,Value=$PROJECT_NAME}]" `
        --region $REGION
    }
    ```

=== "Linux (Bash) :simple-linux:"

    ``` bash linenums="1" hl_lines="1 2 3 4 5 8 9"
    VPC_ID=""
    SUBNET_IDS=""
    SECURITY_GROUP_ID=""
    PROJECT_NAME=""
    REGION=""

    ENDPOINT_LIST=(
        "sts-ep sts"
        "monitoring-ep monitoring"
    )

    for item in "${ENDPOINT_LIST[@]}"
    do
        read -ra list <<< "$item"

        aws ec2 create-vpc-endpoint \
            --vpc-endpoint-type Interface \
            --vpc-id $VPC_ID \
            --service-name com.amazonaws.$REGION.${list[1]} \
            --subnet-ids $SUBNET_IDS \
            --security-group-ids $SECURITY_GROUP_ID \
            --tag-specifications "ResourceType=vpc-endpoint,Tags=[{Key=Name,Value=${list[0]}},{Key=project,Value=$PROJECT_NAME}]" \
            --region $REGION
    done
    ```

## Endpoints List

[AWS Documentation](https://docs.aws.amazon.com/vpc/latest/privatelink/aws-services-privatelink-support.html)

<table>
<thead>
  <tr>
    <th>AWS service</th>
    <th>Service name</th>
    <th>Service name (short)</th>
  </tr>
</thead>
<tbody>
  <tr>
    <td>Access Analyzer</td>
    <td>com.amazonaws.<span style="font-style: italic;">region</span>.access-analyzer</td>
    <td>access-analyzer</td>
  </tr>
  <tr>
    <td><a target="_blank" href="https://docs.aws.amazon.com/accounts/latest/reference/security-privatelink.html">AWS Account Management</a></td>
    <td>com.amazonaws.<span style="font-style: italic;">region</span>.account</td>
    <td>account</td>
  </tr>
  <tr>
    <td><a target="_blank" href="https://docs.aws.amazon.com/apigateway/latest/developerguide/apigateway-private-apis.html">Amazon API Gateway</a></td>
    <td>com.amazonaws.<span style="font-style: italic;">region</span>.execute-api</td>
    <td>execute-api</td>
  </tr>
  <tr>
    <td><a target="_blank" href="https://docs.aws.amazon.com/app-mesh/latest/userguide/vpc-endpoints.html">AWS App Mesh</a></td>
    <td>com.amazonaws.<span style="font-style: italic;">region</span>.appmesh-envoy-management</td>
    <td>appmesh-envoy-management</td>
  </tr>
  <tr>
    <td><a target="_blank" href="https://docs.aws.amazon.com/apprunner/latest/dg/security-vpce.html">AWS App Runner</a></td>
    <td>com.amazonaws.<span style="font-style: italic;">region</span>.apprunner</td>
    <td>apprunner</td>
  </tr>
  <tr>
    <td><a target="_blank" href="https://docs.aws.amazon.com/apprunner/latest/dg/network-pl.html">AWS App Runner services</a></td>
    <td>com.amazonaws.<span style="font-style: italic;">region</span>.apprunner.requests</td>
    <td>apprunner.requests</td>
  </tr>
  <tr>
    <td><a target="_blank" href="https://docs.aws.amazon.com/autoscaling/application/userguide/application-auto-scaling-vpc-endpoints.html">Application Auto Scaling</a></td>
    <td>com.amazonaws.<span style="font-style: italic;">region</span>.application-autoscaling</td>
    <td>application-autoscaling</td>
  </tr>
  <tr>
    <td><a target="_blank" href="https://docs.aws.amazon.com/mgn/latest/ug/AWS-Related-FAQ.html#mgn-and-vpc">AWS Application Migration Service</a></td>
    <td>com.amazonaws.<span style="font-style: italic;">region</span>.mgn</td>
    <td>mgn</td>
  </tr>
  <tr>
    <td rowspan="2"><a target="_blank" href="https://docs.aws.amazon.com/appstream2/latest/developerguide/creating-streaming-from-interface-vpc-endpoints.html">Amazon AppStream 2.0</a></td>
    <td>com.amazonaws.<span style="font-style: italic;">region</span>.appstream.api</td>
    <td>appstream.api</td>
  </tr>
  <tr>
    <td>com.amazonaws.<span style="font-style: italic;">region</span>.appstream.streaming</td>
    <td>appstream.streaming</td>
  </tr>
  <tr>
    <td><a target="_blank" href="https://docs.aws.amazon.com/athena/latest/ug/interface-vpc-endpoint.html">Amazon Athena</a></td>
    <td>com.amazonaws.<span style="font-style: italic;">region</span>.athena</td>
    <td>athena</td>
  </tr>
  <tr>
    <td><a target="_blank" href="https://docs.aws.amazon.com/audit-manager/latest/userguide/vpc-interface-endpoints.html">AWS Audit Manager</a></td>
    <td>com.amazonaws.<span style="font-style: italic;">region</span>.auditmanager</td>
    <td>auditmanager</td>
  </tr>
  <tr>
    <td><a target="_blank" href="https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/vpc-interface-endpoints.html">Amazon Aurora</a></td>
    <td>com.amazonaws.<span style="font-style: italic;">region</span>.rds</td>
    <td>rds</td>
  </tr>
  <tr>
    <td><a target="_blank" href="https://docs.aws.amazon.com/autoscaling/plans/userguide/aws-auto-scaling-vpc-endpoints.html">AWS Auto Scaling</a></td>
    <td>com.amazonaws.<span style="font-style: italic;">region</span>.autoscaling-plans</td>
    <td>autoscaling-plans</td>
  </tr>
  <tr>
    <td rowspan="2"><a target="_blank" href="https://docs.aws.amazon.com/aws-backup/latest/devguide/backup-privatelink.html">AWS Backup</a></td>
    <td>com.amazonaws.<span style="font-style: italic;">region</span>.backup</td>
    <td>backup</td>
  </tr>
  <tr>
    <td>com.amazonaws.<span style="font-style: italic;">region</span>.backup-gateway</td>
    <td>backup-gateway</td>
  </tr>
  <tr>
    <td><a target="_blank" href="https://docs.aws.amazon.com/batch/latest/userguide/vpc-interface-endpoints.html">AWS Batch</a></td>
    <td>com.amazonaws.<span style="font-style: italic;">region</span>.batch</td>
    <td>batch</td>
  </tr>
  <tr>
    <td>AWS Billing Conductor</td>
    <td>com.amazonaws.<span style="font-style: italic;">region</span>.billingconductor</td>
    <td>billingconductor</td>
  </tr>
  <tr>
    <td><a target="_blank" href="https://docs.aws.amazon.com/braket/latest/developerguide/braket-privatelink.html">Amazon Braket</a></td>
    <td>com.amazonaws.<span style="font-style: italic;">region</span>.braket</td>
    <td>braket</td>
  </tr>
  <tr>
    <td><a target="_blank" href="https://docs.aws.amazon.com/privateca/latest/userguide/vpc-endpoints.html">AWS Private Certificate Authority</a></td>
    <td>com.amazonaws.<span style="font-style: italic;">region</span>.acm-pca</td>
    <td>acm-pca</td>
  </tr>
  <tr>
    <td rowspan="2"><a target="_blank" href="https://docs.aws.amazon.com/cloudcontrolapi/latest/userguide/vpc-interface-endpoints.html">AWS Cloud Control API</a></td>
    <td>com.amazonaws.<span style="font-style: italic;">region</span>.cloudcontrolapi</td>
    <td>cloudcontrolapi</td>
  </tr>
  <tr>
    <td>com.amazonaws.<span style="font-style: italic;">region</span>.cloudcontrolapi-fips</td>
    <td>cloudcontrolapi-fips</td>
  </tr>
  <tr>
    <td><a target="_blank" href="https://docs.aws.amazon.com/clouddirectory/latest/developerguide/getting_started_using_vpc_endpoints.html">Amazon Cloud Directory</a></td>
    <td>com.amazonaws.<span style="font-style: italic;">region</span>.clouddirectory</td>
    <td>clouddirectory</td>
  </tr>
  <tr>
    <td><a target="_blank" href="https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/cfn-vpce-bucketnames.html">AWS CloudFormation</a></td>
    <td>com.amazonaws.<span style="font-style: italic;">region</span>.cloudformation</td>
    <td>cloudformation</td>
  </tr>
  <tr>
    <td><a target="_blank" href="https://docs.aws.amazon.com/cloudhsm/latest/userguide/cloudhsm-vpc-endpoint.html">AWS CloudHSM</a></td>
    <td>com.amazonaws.<span style="font-style: italic;">region</span>.cloudhsmv2</td>
    <td>cloudhsmv2</td>
  </tr>
  <tr>
    <td><a target="_blank" href="https://docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudtrail-and-interface-VPC.html">AWS CloudTrail</a></td>
    <td>com.amazonaws.<span style="font-style: italic;">region</span>.cloudtrail</td>
    <td>cloudtrail</td>
  </tr>
  <tr>
    <td rowspan="6"><a target="_blank" href="https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/cloudwatch-and-interface-VPC.html">Amazon CloudWatch</a></td>
    <td>com.amazonaws.<span style="font-style: italic;">region</span>.evidently</td>
    <td>evidently</td>
  </tr>
  <tr>
    <td>com.amazonaws.<span style="font-style: italic;">region</span>.evidently-dataplane</td>
    <td>evidently-dataplane</td>
  </tr>
  <tr>
    <td>com.amazonaws.<span style="font-style: italic;">region</span>.monitoring</td>
    <td>monitoring</td>
  </tr>
  <tr>
    <td>com.amazonaws.<span style="font-style: italic;">region</span>.rum</td>
    <td>rum</td>
  </tr>
  <tr>
    <td>com.amazonaws.<span style="font-style: italic;">region</span>.rum-dataplane</td>
    <td>rum-dataplane</td>
  </tr>
  <tr>
    <td>com.amazonaws.<span style="font-style: italic;">region</span>.synthetics</td>
    <td>synthetics</td>
  </tr>
  <tr>
    <td><a target="_blank" href="https://docs.aws.amazon.com/AmazonCloudWatch/latest/events/cloudwatch-events-and-interface-VPC.html">Amazon CloudWatch Events</a></td>
    <td>com.amazonaws.<span style="font-style: italic;">region</span>.events</td>
    <td>events</td>
  </tr>
  <tr>
    <td><a target="_blank" href="https://docs.aws.amazon.com/AmazonCloudWatch/latest/logs/cloudwatch-logs-and-interface-VPC.html">Amazon CloudWatch Logs</a></td>
    <td>com.amazonaws.<span style="font-style: italic;">region</span>.logs</td>
    <td>logs</td>
  </tr>
  <tr>
    <td rowspan="2"><a target="_blank" href="https://docs.aws.amazon.com/codeartifact/latest/ug/vpc-endpoints.html">AWS CodeArtifact</a></td>
    <td>com.amazonaws.<span style="font-style: italic;">region</span>.codeartifact.api</td>
    <td>codeartifact.api</td>
  </tr>
  <tr>
    <td>com.amazonaws.<span style="font-style: italic;">region</span>.codeartifact.repositories</td>
    <td>codeartifact.repositories</td>
  </tr>
  <tr>
    <td rowspan="2"><a target="_blank" href="https://docs.aws.amazon.com/codebuild/latest/userguide/use-vpc-endpoints-with-codebuild.html">AWS CodeBuild</a></td>
    <td>com.amazonaws.<span style="font-style: italic;">region</span>.codebuild</td>
    <td>codebuild</td>
  </tr>
  <tr>
    <td>com.amazonaws.<span style="font-style: italic;">region</span>.codebuild-fips</td>
    <td>codebuild-fips</td>
  </tr>
  <tr>
    <td rowspan="4"><a target="_blank" href="https://docs.aws.amazon.com/codecommit/latest/userguide/codecommit-and-interface-VPC.html">AWS CodeCommit</a></td>
    <td>com.amazonaws.<span style="font-style: italic;">region</span>.codecommit</td>
    <td>codecommit</td>
  </tr>
  <tr>
    <td>com.amazonaws.<span style="font-style: italic;">region</span>.codecommit-fips</td>
    <td>codecommit-fips</td>
  </tr>
  <tr>
    <td>com.amazonaws.<span style="font-style: italic;">region</span>.git-codecommit</td>
    <td>git-codecommit</td>
  </tr>
  <tr>
    <td>com.amazonaws.<span style="font-style: italic;">region</span>.git-codecommit-fips</td>
    <td>git-codecommit-fips</td>
  </tr>
  <tr>
    <td rowspan="2"><a target="_blank" href="https://docs.aws.amazon.com/codedeploy/latest/userguide/vpc-endpoints.html">AWS CodeDeploy</a></td>
    <td>com.amazonaws.<span style="font-style: italic;">region</span>.codedeploy</td>
    <td>codedeploy</td>
  </tr>
  <tr>
    <td>com.amazonaws.<span style="font-style: italic;">region</span>.codedeploy-commands-secure</td>
    <td>codedeploy-commands-secure</td>
  </tr>
  <tr>
    <td><a target="_blank" href="https://docs.aws.amazon.com/codeguru/latest/profiler-ug/private-link.html">Amazon CodeGuru Profiler</a></td>
    <td>com.amazonaws.<span style="font-style: italic;">region</span>.codeguru-profiler</td>
    <td>codeguru-profiler</td>
  </tr>
  <tr>
    <td><a target="_blank" href="https://docs.aws.amazon.com/codeguru/latest/reviewer-ug/vpc-interface-endpoints.html">Amazon CodeGuru Reviewer</a></td>
    <td>com.amazonaws.<span style="font-style: italic;">region</span>.codeguru-reviewer</td>
    <td>codeguru-reviewer</td>
  </tr>
  <tr>
    <td><a target="_blank" href="https://docs.aws.amazon.com/codepipeline/latest/userguide/vpc-support.html#use-vpc-endpoints-with-codepipeline">AWS CodePipeline</a></td>
    <td>com.amazonaws.<span style="font-style: italic;">region</span>.codepipeline</td>
    <td>codepipeline</td>
  </tr>
  <tr>
    <td><a target="_blank" href="https://docs.aws.amazon.com/dtconsole/latest/userguide/vpc-interface-endpoints.html">AWS CodeStar Connections</a></td>
    <td>com.amazonaws.<span style="font-style: italic;">region</span>.codestar-connections.api</td>
    <td>codestar-connections.api</td>
  </tr>
  <tr>
    <td><a target="_blank" href="https://docs.aws.amazon.com/comprehend/latest/dg/vpc-interface-endpoints.html">Amazon Comprehend</a></td>
    <td>com.amazonaws.<span style="font-style: italic;">region</span>.comprehend</td>
    <td>comprehend</td>
  </tr>
  <tr>
    <td><a target="_blank" href="https://docs.aws.amazon.com/comprehend/latest/dg/vpc-interface-endpoints-med.html">Amazon Comprehend Medical</a></td>
    <td>com.amazonaws.<span style="font-style: italic;">region</span>.comprehendmedical</td>
    <td>comprehendmedical</td>
  </tr>
  <tr>
    <td><a target="_blank" href="https://docs.aws.amazon.com/config/latest/developerguide/config-VPC-endpoints.html">AWS Config</a></td>
    <td>com.amazonaws.<span style="font-style: italic;">region</span>.config</td>
    <td>config</td>
  </tr>
  <tr>
    <td rowspan="6"><a target="_blank" href="https://docs.aws.amazon.com/connect/latest/adminguide/vpc-interface-endpoints.html">Amazon Connect</a></td>
    <td>com.amazonaws.<span style="font-style: italic;">region</span>.app-integrations</td>
    <td>app-integrations</td>
  </tr>
  <tr>
    <td>com.amazonaws.<span style="font-style: italic;">region</span>.cases</td>
    <td>cases</td>
  </tr>
  <tr>
    <td>com.amazonaws.<span style="font-style: italic;">region</span>.connect-campaigns</td>
    <td>connect-campaigns</td>
  </tr>
  <tr>
    <td>com.amazonaws.<span style="font-style: italic;">region</span>.profile</td>
    <td>profile</td>
  </tr>
  <tr>
    <td>com.amazonaws.<span style="font-style: italic;">region</span>.voiceid</td>
    <td>voiceid</td>
  </tr>
  <tr>
    <td>com.amazonaws.<span style="font-style: italic;">region</span>.wisdom</td>
    <td>wisdom</td>
  </tr>
  <tr>
    <td><a target="_blank" href="https://docs.aws.amazon.com/data-exchange/latest/userguide/vpc-interface-endpoints.html">AWS Data Exchange</a></td>
    <td>com.amazonaws.<span style="font-style: italic;">region</span>.dataexchange</td>
    <td>dataexchange</td>
  </tr>
  <tr>
    <td rowspan="2"><a target="_blank" href="https://docs.aws.amazon.com/dms/latest/userguide/CHAP_Security.html#infrastructure-security">AWS Database Migration Service</a></td>
    <td>com.amazonaws.<span style="font-style: italic;">region</span>.dms</td>
    <td>dms</td>
  </tr>
  <tr>
    <td>com.amazonaws.<span style="font-style: italic;">region</span>.dms-fips</td>
    <td>dms-fips</td>
  </tr>
  <tr>
    <td><a target="_blank" href="https://docs.aws.amazon.com/datasync/latest/userguide/datasync-in-vpc.html">AWS DataSync</a></td>
    <td>com.amazonaws.<span style="font-style: italic;">region</span>.datasync</td>
    <td>datasync</td>
  </tr>
  <tr>
    <td><a target="_blank" href="https://docs.aws.amazon.com/devops-guru/latest/userguide/vpc-interface-endpoints.html">Amazon DevOps Guru</a></td>
    <td>com.amazonaws.<span style="font-style: italic;">region</span>.devops-guru</td>
    <td>devops-guru</td>
  </tr>
  <tr>
    <td><a target="_blank" href="https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ebs-apis-vpc-endpoints.html">Amazon EBS direct APIs</a></td>
    <td>com.amazonaws.<span style="font-style: italic;">region</span>.ebs</td>
    <td>ebs</td>
  </tr>
  <tr>
    <td><a target="_blank" href="https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/interface-vpc-endpoints.html">Amazon EC2</a></td>
    <td>com.amazonaws.<span style="font-style: italic;">region</span>.ec2</td>
    <td>ec2</td>
  </tr>
  <tr>
    <td><a target="_blank" href="https://docs.aws.amazon.com/autoscaling/ec2/userguide/ec2-auto-scaling-vpc-endpoints.html">Amazon EC2 Auto Scaling</a></td>
    <td>com.amazonaws.<span style="font-style: italic;">region</span>.autoscaling</td>
    <td>autoscaling</td>
  </tr>
  <tr>
    <td><a target="_blank" href="https://docs.aws.amazon.com/imagebuilder/latest/userguide/vpc-interface-endpoints.html">EC2 Image Builder</a></td>
    <td>com.amazonaws.<span style="font-style: italic;">region</span>.imagebuilder</td>
    <td>imagebuilder</td>
  </tr>
  <tr>
    <td rowspan="2"><a target="_blank" href="https://docs.aws.amazon.com/AmazonECR/latest/userguide/vpc-endpoints.html">Amazon ECR</a></td>
    <td>com.amazonaws.<span style="font-style: italic;">region</span>.ecr.api</td>
    <td>ecr.api</td>
  </tr>
  <tr>
    <td>com.amazonaws.<span style="font-style: italic;">region</span>.ecr.dkr</td>
    <td>ecr.dkr</td>
  </tr>
  <tr>
    <td rowspan="3"><a target="_blank" href="https://docs.aws.amazon.com/AmazonECS/latest/developerguide/vpc-endpoints.html">Amazon ECS</a></td>
    <td>com.amazonaws.<span style="font-style: italic;">region</span>.ecs</td>
    <td>ecs</td>
  </tr>
  <tr>
    <td>com.amazonaws.<span style="font-style: italic;">region</span>.ecs-agent</td>
    <td>ecs-agent</td>
  </tr>
  <tr>
    <td>com.amazonaws.<span style="font-style: italic;">region</span>.ecs-telemetry</td>
    <td>ecs-telemetry</td>
  </tr>
  <tr>
    <td><a target="_blank" href="https://docs.aws.amazon.com/eks/latest/userguide/vpc-interface-endpoints.html">Amazon EKS</a></td>
    <td>com.amazonaws.<span style="font-style: italic;">region</span>.eks</td>
    <td>eks</td>
  </tr>
  <tr>
    <td rowspan="2"><a target="_blank" href="https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/vpc-vpce.html">AWS Elastic Beanstalk</a></td>
    <td>com.amazonaws.<span style="font-style: italic;">region</span>.elasticbeanstalk</td>
    <td>elasticbeanstalk</td>
  </tr>
  <tr>
    <td>com.amazonaws.<span style="font-style: italic;">region</span>.elasticbeanstalk-health</td>
    <td>elasticbeanstalk-health</td>
  </tr>
  <tr>
    <td><a target="_blank" href="https://docs.aws.amazon.com/drs/latest/userguide/drs-and-vpc.html">AWS Elastic Disaster Recovery</a></td>
    <td>com.amazonaws.<span style="font-style: italic;">region</span>.drs</td>
    <td>drs</td>
  </tr>
  <tr>
    <td rowspan="2"><a target="_blank" href="https://docs.aws.amazon.com/efs/latest/ug/efs-vpc-endpoints.html">Amazon Elastic File System</a></td>
    <td>com.amazonaws.<span style="font-style: italic;">region</span>.elasticfilesystem</td>
    <td>elasticfilesystem</td>
  </tr>
  <tr>
    <td>com.amazonaws.<span style="font-style: italic;">region</span>.elasticfilesystem-fips</td>
    <td>elasticfilesystem-fips</td>
  </tr>
  <tr>
    <td><a target="_blank" href="https://docs.aws.amazon.com/elastic-inference/latest/developerguide/setting-up-ei.html#eia-privatelink">Amazon Elastic Inference</a></td>
    <td>com.amazonaws.<span style="font-style: italic;">region</span>.elastic-inference.runtime</td>
    <td>elastic-inference.runtime</td>
  </tr>
  <tr>
    <td><a target="_blank" href="https://docs.aws.amazon.com/elasticloadbalancing/latest/userguide/load-balancer-vpc-endpoints.html">Elastic Load Balancing</a></td>
    <td>com.amazonaws.<span style="font-style: italic;">region</span>.elasticloadbalancing</td>
    <td>elasticloadbalancing</td>
  </tr>
  <tr>
    <td rowspan="2"><a target="_blank" href="https://docs.aws.amazon.com/AmazonElastiCache/latest/red-ug/elasticache-privatelink.html">Amazon ElastiCache</a></td>
    <td>com.amazonaws.<span style="font-style: italic;">region</span>.elasticache</td>
    <td>elasticache</td>
  </tr>
  <tr>
    <td>com.amazonaws.<span style="font-style: italic;">region</span>.elasticache-fips</td>
    <td>elasticache-fips</td>
  </tr>
  <tr>
    <td><a target="_blank" href="https://docs.aws.amazon.com/emr/latest/ManagementGuide/interface-vpc-endpoint.html">Amazon EMR</a></td>
    <td>com.amazonaws.<span style="font-style: italic;">region</span>.elasticmapreduce</td>
    <td>elasticmapreduce</td>
  </tr>
  <tr>
    <td><a target="_blank" href="https://docs.aws.amazon.com/emr/latest/EMR-on-EKS-DevelopmentGuide/security-vpc.html">Amazon EMR on EKS</a></td>
    <td>com.amazonaws.<span style="font-style: italic;">region</span>.emr-containers</td>
    <td>emr-containers</td>
  </tr>
  <tr>
    <td>Amazon EMR Serverless</td>
    <td>com.amazonaws.<span style="font-style: italic;">region</span>.emr-serverless</td>
    <td>emr-serverless</td>
  </tr>
  <tr>
    <td><a target="_blank" href="https://docs.aws.amazon.com/eventbridge/latest/userguide/eb-related-service-vpc.html">Amazon EventBridge</a></td>
    <td>com.amazonaws.<span style="font-style: italic;">region</span>.events</td>
    <td>events</td>
  </tr>
  <tr>
    <td><a target="_blank" href="https://docs.aws.amazon.com/fis/latest/userguide/vpc-interface-endpoints.html">AWS Fault Injection Simulator</a></td>
    <td>com.amazonaws.<span style="font-style: italic;">region</span>.fis</td>
    <td>fis</td>
  </tr>
  <tr>
    <td rowspan="2"><a target="_blank" href="https://docs.aws.amazon.com/finspace/latest/userguide/infrastructure-security.html#connect-vpce">Amazon FinSpace</a></td>
    <td>com.amazonaws.<span style="font-style: italic;">region</span>.finspace</td>
    <td>finspace</td>
  </tr>
  <tr>
    <td>com.amazonaws.<span style="font-style: italic;">region</span>.finspace-api</td>
    <td>finspace-api</td>
  </tr>
  <tr>
    <td rowspan="4"><a target="_blank" href="https://docs.aws.amazon.com/forecast/latest/dg/vpc-interface-endpoints.html">Amazon Forecast</a></td>
    <td>com.amazonaws.<span style="font-style: italic;">region</span>.forecast</td>
    <td>forecast</td>
  </tr>
  <tr>
    <td>com.amazonaws.<span style="font-style: italic;">region</span>.forecastquery</td>
    <td>forecastquery</td>
  </tr>
  <tr>
    <td>com.amazonaws.<span style="font-style: italic;">region</span>.forecast-fips</td>
    <td>forecast-fips</td>
  </tr>
  <tr>
    <td>com.amazonaws.<span style="font-style: italic;">region</span>.forecastquery-fips</td>
    <td>forecastquery-fips</td>
  </tr>
  <tr>
    <td><a target="_blank" href="https://docs.aws.amazon.com/frauddetector/latest/ug/vpc-interface-endpoints.html">Amazon Fraud Detector</a></td>
    <td>com.amazonaws.<span style="font-style: italic;">region</span>.frauddetector</td>
    <td>frauddetector</td>
  </tr>
  <tr>
    <td rowspan="2">Amazon FSx</td>
    <td>com.amazonaws.<span style="font-style: italic;">region</span>.fsx</td>
    <td>fsx</td>
  </tr>
  <tr>
    <td>com.amazonaws.<span style="font-style: italic;">region</span>.fsx-fips</td>
    <td>fsx-fips</td>
  </tr>
  <tr>
    <td><a target="_blank" href="https://docs.aws.amazon.com/glue/latest/dg/vpc-interface-endpoints.html">AWS Glue</a></td>
    <td>com.amazonaws.<span style="font-style: italic;">region</span>.glue</td>
    <td>glue</td>
  </tr>
  <tr>
    <td><a target="_blank" href="https://docs.aws.amazon.com/databrew/latest/dg/vpc-endpoint.html">AWS Glue DataBrew</a></td>
    <td>com.amazonaws.<span style="font-style: italic;">region</span>.databrew</td>
    <td>databrew</td>
  </tr>
  <tr>
    <td rowspan="2"><a target="_blank" href="https://docs.aws.amazon.com/grafana/latest/userguide/VPC-endpoints.html">Amazon Managed Grafana</a></td>
    <td>com.amazonaws.<span style="font-style: italic;">region</span>.grafana</td>
    <td>grafana</td>
  </tr>
  <tr>
    <td>com.amazonaws.<span style="font-style: italic;">region</span>.grafana-workspace</td>
    <td>grafana-workspace</td>
  </tr>
  <tr>
    <td>AWS Ground Station</td>
    <td>com.amazonaws.<span style="font-style: italic;">region</span>.groundstation</td>
    <td>groundstation</td>
  </tr>
  <tr>
    <td><a target="_blank" href="https://docs.aws.amazon.com/healthlake/latest/devguide/vpc-endpoints.html">Amazon HealthLake</a></td>
    <td>com.amazonaws.<span style="font-style: italic;">region</span>.healthlake</td>
    <td>healthlake</td>
  </tr>
  <tr>
    <td>IAM Identity Center</td>
    <td>com.amazonaws.<span style="font-style: italic;">region</span>.identitystore</td>
    <td>identitystore</td>
  </tr>
  <tr>
    <td><a target="_blank" href="https://docs.aws.amazon.com/rolesanywhere/latest/userguide/vpc-interface-endpoints.html">IAM Roles Anywhere</a></td>
    <td>com.amazonaws.<span style="font-style: italic;">region</span>.rolesanywhere</td>
    <td>rolesanywhere</td>
  </tr>
  <tr>
    <td>Amazon Inspector</td>
    <td>com.amazonaws.<span style="font-style: italic;">region</span>.inspector2</td>
    <td>inspector2</td>
  </tr>
  <tr>
    <td><a target="_blank" href="https://docs.aws.amazon.com/iot/latest/developerguide/IoTCore-VPC.html">AWS IoT Core</a></td>
    <td>com.amazonaws.<span style="font-style: italic;">region</span>.iot.data</td>
    <td>iot.data</td>
  </tr>
  <tr>
    <td><a target="_blank" href="https://docs.aws.amazon.com/iot/latest/developerguide/device-advisor-vpc-endpoint.html">AWS IoT Core Device Advisor</a></td>
    <td>com.amazonaws.<span style="font-style: italic;">region</span>.deviceadvisor.iot</td>
    <td>deviceadvisor.iot</td>
  </tr>
  <tr>
    <td rowspan="3"><a target="_blank" href="https://docs.aws.amazon.com/iot/latest/developerguide/connect-iot-lorawan-interface-vpc-endpoint.html">AWS IoT Core for LoRaWAN</a></td>
    <td>com.amazonaws.<span style="font-style: italic;">region</span>.iotwireless.api</td>
    <td>iotwireless.api</td>
  </tr>
  <tr>
    <td>com.amazonaws.<span style="font-style: italic;">region</span>.lorawan.cups</td>
    <td>lorawan.cups</td>
  </tr>
  <tr>
    <td>com.amazonaws.<span style="font-style: italic;">region</span>.lorawan.lns</td>
    <td>lorawan.lns</td>
  </tr>
  <tr>
    <td><a target="_blank" href="https://docs.aws.amazon.com/greengrass/v2/developerguide/vpc-interface-endpoints.html">AWS IoT Greengrass</a></td>
    <td>com.amazonaws.<span style="font-style: italic;">region</span>.greengrass</td>
    <td>greengrass</td>
  </tr>
  <tr>
    <td><a target="_blank" href="https://docs.aws.amazon.com/iotroborunner/latest/dev/vpc-interface-endpoints.html">AWS IoT RoboRunner</a></td>
    <td>com.amazonaws.<span style="font-style: italic;">region</span>.iotroborunner</td>
    <td>iotroborunner</td>
  </tr>
  <tr>
    <td rowspan="2"><a target="_blank" href="https://docs.aws.amazon.com/iot-sitewise/latest/userguide/vpc-interface-endpoints.html">AWS IoT SiteWise</a></td>
    <td>com.amazonaws.<span style="font-style: italic;">region</span>.iotsitewise.api</td>
    <td>iotsitewise.api</td>
  </tr>
  <tr>
    <td>com.amazonaws.<span style="font-style: italic;">region</span>.iotsitewise.data</td>
    <td>iotsitewise.data</td>
  </tr>
  <tr>
    <td rowspan="2"><a target="_blank" href="https://docs.aws.amazon.com/iot-twinmaker/latest/guide/vpc-interface-endpoints.html">AWS IoT TwinMaker</a></td>
    <td>com.amazonaws.<span style="font-style: italic;">region</span>.iottwinmaker.api</td>
    <td>iottwinmaker.api</td>
  </tr>
  <tr>
    <td>com.amazonaws.<span style="font-style: italic;">region</span>.iottwinmaker.data</td>
    <td>iottwinmaker.data</td>
  </tr>
  <tr>
    <td rowspan="2"><a target="_blank" href="https://docs.aws.amazon.com/kendra/latest/dg/vpc-interface-endpoints.html">Amazon Kendra</a></td>
    <td>com.amazonaws.<span style="font-style: italic;">region</span>.kendra</td>
    <td>kendra</td>
  </tr>
  <tr>
    <td>aws.api.<span style="font-style: italic;">region</span>.kendra-ranking</td>
    <td></td>
  </tr>
  <tr>
    <td><a target="_blank" href="https://docs.aws.amazon.com/kms/latest/developerguide/kms-vpc-endpoint.html">AWS Key Management Service</a></td>
    <td>com.amazonaws.<span style="font-style: italic;">region</span>.kms</td>
    <td>kms</td>
  </tr>
  <tr>
    <td rowspan="2"><a target="_blank" href="https://docs.aws.amazon.com/keyspaces/latest/devguide/vpc-endpoints.html">Amazon Keyspaces (for Apache Cassandra)</a></td>
    <td>com.amazonaws.<span style="font-style: italic;">region</span>.cassandra</td>
    <td>cassandra</td>
  </tr>
  <tr>
    <td>com.amazonaws.<span style="font-style: italic;">region</span>.cassandra-fips</td>
    <td>cassandra-fips</td>
  </tr>
  <tr>
    <td><a target="_blank" href="https://docs.aws.amazon.com/firehose/latest/dev/vpc.html">Amazon Kinesis Data Firehose</a></td>
    <td>com.amazonaws.<span style="font-style: italic;">region</span>.kinesis-firehose</td>
    <td>kinesis-firehose</td>
  </tr>
  <tr>
    <td><a target="_blank" href="https://docs.aws.amazon.com/streams/latest/dev/vpc.html">Amazon Kinesis Data Streams</a></td>
    <td>com.amazonaws.<span style="font-style: italic;">region</span>.kinesis-streams</td>
    <td>kinesis-streams</td>
  </tr>
  <tr>
    <td><a target="_blank" href="https://docs.aws.amazon.com/lake-formation/latest/dg/privatelink.html">AWS Lake Formation</a></td>
    <td>com.amazonaws.<span style="font-style: italic;">region</span>.lakeformation</td>
    <td>lakeformation</td>
  </tr>
  <tr>
    <td><a target="_blank" href="https://docs.aws.amazon.com/lambda/latest/dg/configuration-vpc-endpoints.html">AWS Lambda</a></td>
    <td>com.amazonaws.<span style="font-style: italic;">region</span>.lambda</td>
    <td>lambda</td>
  </tr>
  <tr>
    <td rowspan="2"><a target="_blank" href="https://docs.aws.amazon.com/lexv2/latest/dg/vpc-interface-endpoints.html">Amazon Lex</a></td>
    <td>com.amazonaws.<span style="font-style: italic;">region</span>.models-v2-lex</td>
    <td>models-v2-lex</td>
  </tr>
  <tr>
    <td>com.amazonaws.<span style="font-style: italic;">region</span>.runtime-v2-lex</td>
    <td>runtime-v2-lex</td>
  </tr>
  <tr>
    <td rowspan="2"><a target="_blank" href="https://docs.aws.amazon.com/license-manager/latest/userguide/interface-vpc-endpoints.html">AWS License Manager</a></td>
    <td>com.amazonaws.<span style="font-style: italic;">region</span>.license-manager</td>
    <td>license-manager</td>
  </tr>
  <tr>
    <td>com.amazonaws.<span style="font-style: italic;">region</span>.license-manager-fips</td>
    <td>license-manager-fips</td>
  </tr>
  <tr>
    <td><a target="_blank" href="https://docs.aws.amazon.com/lookout-for-equipment/latest/ug/vpc-interface-endpoints.html">Amazon Lookout for Equipment</a></td>
    <td>com.amazonaws.<span style="font-style: italic;">region</span>.lookoutequipment</td>
    <td>lookoutequipment</td>
  </tr>
  <tr>
    <td><a target="_blank" href="https://docs.aws.amazon.com/lookoutmetrics/latest/dev/services-vpc.html#services-vpc-interface">Amazon Lookout for Metrics</a></td>
    <td>com.amazonaws.<span style="font-style: italic;">region</span>.lookoutmetrics</td>
    <td>lookoutmetrics</td>
  </tr>
  <tr>
    <td><a target="_blank" href="https://docs.aws.amazon.com/lookout-for-vision/latest/developer-guide/vpc-interface-endpoints.html">Amazon Lookout for Vision</a></td>
    <td>com.amazonaws.<span style="font-style: italic;">region</span>.lookoutvision</td>
    <td>lookoutvision</td>
  </tr>
  <tr>
    <td><a target="_blank" href="https://docs.aws.amazon.com/macie/latest/user/vpc-interface-endpoints.html">Amazon Macie</a></td>
    <td>com.amazonaws.<span style="font-style: italic;">region</span>.macie2</td>
    <td>macie2</td>
  </tr>
  <tr>
    <td><a target="_blank" href="https://docs.aws.amazon.com/m2/latest/userguide/vpc-interface-endpoints.html">AWS Mainframe Modernization</a></td>
    <td>com.amazonaws.<span style="font-style: italic;">region</span>.m2</td>
    <td>m2</td>
  </tr>
  <tr>
    <td rowspan="2"><a target="_blank" href="https://docs.aws.amazon.com/prometheus/latest/userguide/AMP-and-interface-VPC.html">Amazon Managed Service for Prometheus</a></td>
    <td>com.amazonaws.<span style="font-style: italic;">region</span>.aps</td>
    <td>aps</td>
  </tr>
  <tr>
    <td>com.amazonaws.<span style="font-style: italic;">region</span>.aps-workspaces</td>
    <td>aps-workspaces</td>
  </tr>
  <tr>
    <td rowspan="3"><a target="_blank" href="https://docs.aws.amazon.com/mwaa/latest/userguide/vpc-vpe-create-access.html">Amazon Managed Workflows for Apache Airflow</a></td>
    <td>com.amazonaws.<span style="font-style: italic;">region</span>.airflow.api</td>
    <td>airflow.api</td>
  </tr>
  <tr>
    <td>com.amazonaws.<span style="font-style: italic;">region</span>.airflow.env</td>
    <td>airflow.env</td>
  </tr>
  <tr>
    <td>com.amazonaws.<span style="font-style: italic;">region</span>.airflow.ops</td>
    <td>airflow.ops</td>
  </tr>
  <tr>
    <td rowspan="2"><a target="_blank" href="https://docs.aws.amazon.com/memorydb/latest/devguide/memorydb-privatelink.html">Amazon MemoryDB for Redis</a></td>
    <td>com.amazonaws.<span style="font-style: italic;">region</span>.memory-db</td>
    <td>memory-db</td>
  </tr>
  <tr>
    <td>com.amazonaws.<span style="font-style: italic;">region</span>.memorydb-fips</td>
    <td>memorydb-fips</td>
  </tr>
  <tr>
    <td><a target="_blank" href="https://docs.aws.amazon.com/migrationhub-orchestrator/latest/userguide/vpc-interface-endpoints.html">AWS Migration Hub Orchestrator</a></td>
    <td>com.amazonaws.<span style="font-style: italic;">region</span>.migrationhub-orchestrator</td>
    <td>migrationhub-orchestrator</td>
  </tr>
  <tr>
    <td><a target="_blank" href="https://docs.aws.amazon.com/migrationhub-refactor-spaces/latest/userguide/vpc-interface-endpoints.html">AWS Migration Hub Refactor Spaces</a></td>
    <td>com.amazonaws.<span style="font-style: italic;">region</span>.refactor-spaces</td>
    <td>refactor-spaces</td>
  </tr>
  <tr>
    <td><a target="_blank" href="https://docs.aws.amazon.com/migrationhub-strategy/latest/userguide/vpc-interface-endpoints.html">Migration Hub Strategy Recommendations</a></td>
    <td>com.amazonaws.<span style="font-style: italic;">region</span>.migrationhub-strategy</td>
    <td>migrationhub-strategy</td>
  </tr>
  <tr>
    <td>Amazon Nimble Studio</td>
    <td>com.amazonaws.<span style="font-style: italic;">region</span>.nimble</td>
    <td>nimble</td>
  </tr>
  <tr>
    <td rowspan="5"><a target="_blank" href="https://docs.aws.amazon.com/omics/latest/dev/vpc-interface-endpoints.html">Amazon Omics</a></td>
    <td>com.amazonaws.<span style="font-style: italic;">region</span>.analytics-omics</td>
    <td>analytics-omics</td>
  </tr>
  <tr>
    <td>com.amazonaws.<span style="font-style: italic;">region</span>.control-storage-omics</td>
    <td>control-storage-omics</td>
  </tr>
  <tr>
    <td>com.amazonaws.<span style="font-style: italic;">region</span>.storage-omics</td>
    <td>storage-omics</td>
  </tr>
  <tr>
    <td>com.amazonaws.<span style="font-style: italic;">region</span>.tags-omics</td>
    <td>tags-omics</td>
  </tr>
  <tr>
    <td>com.amazonaws.<span style="font-style: italic;">region</span>.workflows-omics</td>
    <td>workflows-omics</td>
  </tr>
  <tr>
    <td><a target="_blank" href="https://docs.aws.amazon.com/opensearch-service/latest/developerguide/vpc-interface-endpoints.html">Amazon OpenSearch Service</a></td>
    <td>These endpoints are service-managed</td>
    <td></td>
  </tr>
  <tr>
    <td><a target="_blank" href="https://docs.aws.amazon.com/panorama/latest/dev/api-endpoints.html">AWS Panorama</a></td>
    <td>com.amazonaws.<span style="font-style: italic;">region</span>.panorama</td>
    <td>panorama</td>
  </tr>
  <tr>
    <td>Amazon Pinpoint</td>
    <td>com.amazonaws.<span style="font-style: italic;">region</span>.pinpoint-sms-voice-v2</td>
    <td>pinpoint-sms-voice-v2</td>
  </tr>
  <tr>
    <td><a target="_blank" href="https://docs.aws.amazon.com/polly/latest/dg/using-polly-with-vpc-endpoints.html">Amazon Polly</a></td>
    <td>com.amazonaws.<span style="font-style: italic;">region</span>.polly</td>
    <td>polly</td>
  </tr>
  <tr>
    <td>AWS Private 5G</td>
    <td>com.amazonaws.<span style="font-style: italic;">region</span>.private-networks</td>
    <td>private-networks</td>
  </tr>
  <tr>
    <td><a target="_blank" href="https://docs.aws.amazon.com/proton/latest/userguide/infrastructure-security.html#vpc-interface-endpoints">AWS Proton</a></td>
    <td>com.amazonaws.<span style="font-style: italic;">region</span>.proton</td>
    <td>proton</td>
  </tr>
  <tr>
    <td><a target="_blank" href="https://docs.aws.amazon.com/qldb/latest/developerguide/vpc-endpoints.html#using-interface-vpc-endpoints">Amazon QLDB</a></td>
    <td>com.amazonaws.<span style="font-style: italic;">region</span>.qldb.session</td>
    <td>qldb.session</td>
  </tr>
  <tr>
    <td><a target="_blank" href="https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/vpc-interface-endpoints.html">Amazon RDS</a></td>
    <td>com.amazonaws.<span style="font-style: italic;">region</span>.rds</td>
    <td>rds</td>
  </tr>
  <tr>
    <td><a target="_blank" href="https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/data-api.html#data-api.vpc-endpoint">Amazon RDS Data API</a></td>
    <td>com.amazonaws.<span style="font-style: italic;">region</span>.rds-data</td>
    <td>rds-data</td>
  </tr>
  <tr>
    <td rowspan="2"><a target="_blank" href="https://docs.aws.amazon.com/redshift/latest/mgmt/security-private-link.html">Amazon Redshift</a></td>
    <td>com.amazonaws.<span style="font-style: italic;">region</span>.redshift</td>
    <td>redshift</td>
  </tr>
  <tr>
    <td>com.amazonaws.<span style="font-style: italic;">region</span>.redshift-fips</td>
    <td>redshift-fips</td>
  </tr>
  <tr>
    <td><a target="_blank" href="https://docs.aws.amazon.com/redshift/latest/mgmt/data-api-access.html#data-api-vpc-endpoint">Amazon Redshift Data API</a></td>
    <td>com.amazonaws.<span style="font-style: italic;">region</span>.redshift-data</td>
    <td>redshift-data</td>
  </tr>
  <tr>
    <td rowspan="2"><a target="_blank" href="https://docs.aws.amazon.com/rekognition/latest/dg/vpc.html">Amazon Rekognition</a></td>
    <td>com.amazonaws.<span style="font-style: italic;">region</span>.rekognition</td>
    <td>rekognition</td>
  </tr>
  <tr>
    <td>com.amazonaws.<span style="font-style: italic;">region</span>.rekognition-fips</td>
    <td>rekognition-fips</td>
  </tr>
  <tr>
    <td><a target="_blank" href="https://docs.aws.amazon.com/robomaker/latest/dg/vpc-interface-endpoints.html">AWS RoboMaker</a></td>
    <td>com.amazonaws.<span style="font-style: italic;">region</span>.robomaker</td>
    <td>robomaker</td>
  </tr>
  <tr>
    <td><a target="_blank" href="https://docs.aws.amazon.com/AmazonS3/latest/userguide/privatelink-interface-endpoints.html">Amazon S3</a></td>
    <td>com.amazonaws.<span style="font-style: italic;">region</span>.s3</td>
    <td>s3</td>
  </tr>
  <tr>
    <td><a target="_blank" href="https://docs.aws.amazon.com/AmazonS3/latest/userguide/MultiRegionAccessPointsPrivateLink.html">Amazon S3 Multi-Region Access Points</a></td>
    <td>com.amazonaws.s3-global.accesspoint</td>
    <td></td>
  </tr>
  <tr>
    <td><a target="_blank" href="https://docs.aws.amazon.com/AmazonS3/latest/userguide/s3-outposts-privatelink-interface-endpoints.html">Amazon S3 on Outposts</a></td>
    <td>com.amazonaws.<span style="font-style: italic;">region</span>.s3-outposts</td>
    <td>s3-outposts</td>
  </tr>
  <tr>
    <td rowspan="7"><a target="_blank" href="https://docs.aws.amazon.com/sagemaker/latest/dg/notebook-interface-endpoint.html">Amazon SageMaker</a></td>
    <td>aws.sagemaker.<span style="font-style: italic;">region</span>.notebook</td>
    <td></td>
  </tr>
  <tr>
    <td>aws.sagemaker.<span style="font-style: italic;">region</span>.studio</td>
    <td></td>
  </tr>
  <tr>
    <td>com.amazonaws.<span style="font-style: italic;">region</span>.sagemaker.api</td>
    <td>sagemaker.api</td>
  </tr>
  <tr>
    <td>com.amazonaws.<span style="font-style: italic;">region</span>.sagemaker.featurestore-runtime</td>
    <td>sagemaker.featurestore-runtime</td>
  </tr>
  <tr>
    <td>com.amazonaws.<span style="font-style: italic;">region</span>.sagemaker.metrics</td>
    <td>sagemaker.metrics</td>
  </tr>
  <tr>
    <td>com.amazonaws.<span style="font-style: italic;">region</span>.sagemaker.runtime</td>
    <td>sagemaker.runtime</td>
  </tr>
  <tr>
    <td>com.amazonaws.<span style="font-style: italic;">region</span>.sagemaker.runtime-fips</td>
    <td>sagemaker.runtime-fips</td>
  </tr>
  <tr>
    <td><a target="_blank" href="https://docs.aws.amazon.com/secretsmanager/latest/userguide/vpc-endpoint-overview.html">AWS Secrets Manager</a></td>
    <td>com.amazonaws.<span style="font-style: italic;">region</span>.secretsmanager</td>
    <td>secretsmanager</td>
  </tr>
  <tr>
    <td><a target="_blank" href="https://docs.aws.amazon.com/securityhub/latest/userguide/security-vpc-endpoints.html">AWS Security Hub</a></td>
    <td>com.amazonaws.<span style="font-style: italic;">region</span>.securityhub</td>
    <td>securityhub</td>
  </tr>
  <tr>
    <td><a target="_blank" href="https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_sts_vpce.html">AWS Security Token Service</a></td>
    <td>com.amazonaws.<span style="font-style: italic;">region</span>.sts</td>
    <td>sts</td>
  </tr>
  <tr>
    <td rowspan="3">AWS Server Migration Service</td>
    <td>com.amazonaws.<span style="font-style: italic;">region</span>.awsconnector</td>
    <td>awsconnector</td>
  </tr>
  <tr>
    <td>com.amazonaws.<span style="font-style: italic;">region</span>.sms</td>
    <td>sms</td>
  </tr>
  <tr>
    <td>com.amazonaws.<span style="font-style: italic;">region</span>.sms-fips</td>
    <td>sms-fips</td>
  </tr>
  <tr>
    <td rowspan="2">Service Catalog</td>
    <td>com.amazonaws.<span style="font-style: italic;">region</span>.servicecatalog</td>
    <td>servicecatalog</td>
  </tr>
  <tr>
    <td>com.amazonaws.<span style="font-style: italic;">region</span>.servicecatalog-appregistry</td>
    <td>servicecatalog-appregistry</td>
  </tr>
  <tr>
    <td><a target="_blank" href="https://docs.aws.amazon.com/ses/latest/DeveloperGuide/send-email-set-up-vpc-endpoints.html">Amazon SES</a></td>
    <td>com.amazonaws.<span style="font-style: italic;">region</span>.email-smtp</td>
    <td>email-smtp</td>
  </tr>
  <tr>
    <td>AWS Snow Device Management</td>
    <td>com.amazonaws.<span style="font-style: italic;">region</span>.snow-device-management</td>
    <td>snow-device-management</td>
  </tr>
  <tr>
    <td><a target="_blank" href="https://docs.aws.amazon.com/sns/latest/dg/sns-vpc.html">Amazon SNS</a></td>
    <td>com.amazonaws.<span style="font-style: italic;">region</span>.sns</td>
    <td>sns</td>
  </tr>
  <tr>
    <td><a target="_blank" href="https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/sqs-internetwork-traffic-privacy.html#sqs-vpc-endpoints">Amazon SQS</a></td>
    <td>com.amazonaws.<span style="font-style: italic;">region</span>.sqs</td>
    <td>sqs</td>
  </tr>
  <tr>
    <td rowspan="2"><a target="_blank" href="https://docs.aws.amazon.com/step-functions/latest/dg/vpc-endpoints.html">AWS Step Functions</a></td>
    <td>com.amazonaws.<span style="font-style: italic;">region</span>.states</td>
    <td>states</td>
  </tr>
  <tr>
    <td>com.amazonaws.<span style="font-style: italic;">region</span>.sync-states</td>
    <td>sync-states</td>
  </tr>
  <tr>
    <td>AWS Storage Gateway</td>
    <td>com.amazonaws.<span style="font-style: italic;">region</span>.storagegateway</td>
    <td>storagegateway</td>
  </tr>
  <tr>
    <td rowspan="5"><a target="_blank" href="https://docs.aws.amazon.com/systems-manager/latest/userguide/sysman-setting-up-vpc.html">AWS Systems Manager</a></td>
    <td>com.amazonaws.<span style="font-style: italic;">region</span>.ec2messages</td>
    <td>ec2messages</td>
  </tr>
  <tr>
    <td>com.amazonaws.<span style="font-style: italic;">region</span>.ssm</td>
    <td>ssm</td>
  </tr>
  <tr>
    <td>com.amazonaws.<span style="font-style: italic;">region</span>.ssm-contacts</td>
    <td>ssm-contacts</td>
  </tr>
  <tr>
    <td>com.amazonaws.<span style="font-style: italic;">region</span>.ssm-incidents</td>
    <td>ssm-incidents</td>
  </tr>
  <tr>
    <td>com.amazonaws.<span style="font-style: italic;">region</span>.ssmmessages</td>
    <td>ssmmessages</td>
  </tr>
  <tr>
    <td rowspan="2"><a target="_blank" href="https://docs.aws.amazon.com/textract/latest/dg/vpc-interface-endpoints.html">Amazon Textract</a></td>
    <td>com.amazonaws.<span style="font-style: italic;">region</span>.textract</td>
    <td>textract</td>
  </tr>
  <tr>
    <td>com.amazonaws.<span style="font-style: italic;">region</span>.textract-fips</td>
    <td>textract-fips</td>
  </tr>
  <tr>
    <td rowspan="2"><a target="_blank" href="https://docs.aws.amazon.com/transcribe/latest/dg/vpc-interface-endpoints.html">Amazon Transcribe</a></td>
    <td>com.amazonaws.<span style="font-style: italic;">region</span>.transcribe</td>
    <td>transcribe</td>
  </tr>
  <tr>
    <td>com.amazonaws.<span style="font-style: italic;">region</span>.transcribestreaming</td>
    <td>transcribestreaming</td>
  </tr>
  <tr>
    <td rowspan="2"><a target="_blank" href="https://docs.aws.amazon.com/transcribe/latest/dg/med-vpc-interface-endpoints.html">Amazon Transcribe Medical</a></td>
    <td>com.amazonaws.<span style="font-style: italic;">region</span>.transcribe</td>
    <td>transcribe</td>
  </tr>
  <tr>
    <td>com.amazonaws.<span style="font-style: italic;">region</span>.transcribestreaming</td>
    <td>transcribestreaming</td>
  </tr>
  <tr>
    <td rowspan="2">AWS Transfer for SFTP</td>
    <td>com.amazonaws.<span style="font-style: italic;">region</span>.transfer</td>
    <td>transfer</td>
  </tr>
  <tr>
    <td>com.amazonaws.<span style="font-style: italic;">region</span>.transfer.server</td>
    <td>transfer.server</td>
  </tr>
  <tr>
    <td><a target="_blank" href="https://docs.aws.amazon.com/translate/latest/dg/vpc-interface-endpoints.html">Amazon Translate</a></td>
    <td>com.amazonaws.<span style="font-style: italic;">region</span>.translate</td>
    <td>translate</td>
  </tr>
  <tr>
    <td><a target="_blank" href="https://docs.aws.amazon.com/workspaces/latest/adminguide/infrastructure-security.html#interface-vpc-endpoint">Amazon WorkSpaces</a></td>
    <td>com.amazonaws.<span style="font-style: italic;">region</span>.workspaces</td>
    <td>workspaces</td>
  </tr>
  <tr>
    <td><a target="_blank" href="https://docs.aws.amazon.com/xray/latest/devguide/xray-security-vpc-endpoint.html">AWS X-Ray</a></td>
    <td>com.amazonaws.<span style="font-style: italic;">region</span>.xray</td>
    <td>xray</td>
  </tr>
</tbody>
</table>