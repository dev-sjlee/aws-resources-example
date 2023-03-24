# Install Amazon Managed Prometheus

!!! note

    Maybe you need EBS CSI Driver to use AMP. [Using EBS](../15-using-ebs/)

## Create prometheus workspace

=== ":simple-linux: Linux"

    ``` bash hl_lines="1 2 3 5 6"
    STACK_NAME="<stack name>"
    PROJECT_NAME="<project name>"
    REGION="<region code>"
    
    WORKSPACE_NAME=""                             # [REQUIRED] The name of this APS workspace.
    LOG_GROUP_NAME="/aws/vendedlogs/prometheus"   # [REQUIRED] The name of this APS workspace's log group.

    curl -LO https://raw.githubusercontent.com/marcus16-kang/cloudformation-templates/main/aps/workspace.yaml

    aws cloudformation deploy \
        --stack-name $STACK_NAME \
        --template-file ./workspace.yaml \
        --parameter-overrides \
            ProjectName=$PROJECT_NAME \
            WorkspaceName=$WORKSPACE_NAME \
            LogGroupName=$LOG_GROUP_NAME \
        --tags project=$PROJECT_NAME \
        --region $REGION \
        --disable-rollback
    ```

=== ":simple-windows: Windows"

    ``` powershell hl_lines="1 2 3 5 6"
    $STACK_NAME="<stack name>"
    $PROJECT_NAME="<project name>"
    $REGION="<region code>"
    
    $WORKSPACE_NAME=""                            # [REQUIRED] The name of this APS workspace.
    $LOG_GROUP_NAME="/aws/vendedlogs/prometheus"  # [REQUIRED] The name of this APS workspace's log group.

    curl.exe -LO https://raw.githubusercontent.com/marcus16-kang/cloudformation-templates/main/aps/workspace.yaml

    aws cloudformation deploy `
        --stack-name $STACK_NAME `
        --template-file ./workspace.yaml `
        --parameter-overrides `
            ProjectName=$PROJECT_NAME `
            WorkspaceName=$WORKSPACE_NAME `
            LogGroupName=$LOG_GROUP_NAME `
        --tags project=$PROJECT_NAME `
        --region $REGION `
        --disable-rollback
    ```

[AWS Documentation](https://awscli.amazonaws.com/v2/documentation/api/latest/reference/amp/create-workspace.html)

## Create prometheus namespace

``` shell
kubectl create namespace prometheus
```

## Create prometheus IAM role for service account

=== ":simple-linux: Linux"

    ``` bash hl_lines="1 2 3 4 5"
    CLUSTER_NAME="<cluster name>"
    STACK_NAME="<workspace stack name>"
    POLICY_NAME="<policy name>"
    ROLE_NAME="<role name>"
    REGION="<region code>"

    AMP_WORKSPACE_ARN=$(aws cloudformation describe-stacks \
        --stack-name $STACK_NAME \
        --query 'Stacks[0].Outputs[?OutputKey==`WorkspaceArn`].OutputValue' \
        --region $REGION \
        --output text
    )

    curl -LO https://raw.githubusercontent.com/marcus16-kang/aws-resources-example/main/scripts/eks/prometheus-policy.json
    sed -i "s|AMP_WORKSPACE_ARN|$AMP_WORKSPACE_ARN|" ./prometheus-policy.json

    POLICY_ARN=$(aws iam create-policy \
        --policy-name $POLICY_NAME \
        --policy-document file://prometheus-policy.json \
        --query 'Policy.Arn' \
        --output text \
        # --tags Key=project,Value=$PROJECT_NAME \  # AWS CLI v2
    )

    eksctl create iamserviceaccount \
        --cluster $CLUSTER_NAME \
        --namespace prometheus \
        --name prometheus-server \
        --role-name $ROLE_NAME \
        --attach-policy-arn $POLICY_ARN \
        --tags project=$PROJECT_NAME \
        --region $REGION \
        --override-existing-serviceaccounts \
        --approve
    ```

=== ":simple-windows: Windows"

    ``` powershell hl_lines="1 2 3 4 5"
    $CLUSTER_NAME="<cluster name>"
    $STACK_NAME="<workspace stack name>"
    $POLICY_NAME="<policy name>"
    $ROLE_NAME="<role name>"
    $REGION="<region code>"

    $AMP_WORKSPACE_ARN = aws cloudformation describe-stacks `
        --stack-name $STACK_NAME `
        --query 'Stacks[0].Outputs[?OutputKey==`WorkspaceArn`].OutputValue' `
        --region $REGION `
        --output text

    curl.exe -LO https://raw.githubusercontent.com/marcus16-kang/aws-resources-example/main/scripts/eks/prometheus-policy.json
    $content = Get-Content ./prometheus-policy.json
    $content = $content -replace "AMP_WORKSPACE_ARN", $AMP_WORKSPACE_ARN
    $content | Set-Content ./prometheus-policy.json

    POLICY_ARN = aws iam create-policy `
        --policy-name $POLICY_NAME `
        --policy-document file://prometheus-policy.json `
        --query 'Policy.Arn' `
        --output text `
        --tags Key=project,Value=$PROJECT_NAME

    eksctl create iamserviceaccount `
        --cluster $CLUSTER_NAME `
        --namespace prometheus `
        --name prometheus-server `
        --role-name $ROLE_NAME `
        --attach-policy-arn $POLICY_ARN `
        --tags project=$PROJECT_NAME `
        --region $REGION `
        --override-existing-serviceaccounts `
        --approve
    ```

[AWS Documentation](https://docs.aws.amazon.com/prometheus/latest/userguide/set-up-irsa.html#set-up-irsa-ingest)

## Install prometheus with `helm`

=== ":simple-linux: Linux"

    ``` shell hl_lines="1 2"
    STACK_NAME="<workspace stack name>"
    REGION="<region code>"

    APS_ENDPOINT=$(aws cloudformation describe-stacks \
        --stack-name $STACK_NAME \
        --query 'Stacks[0].Outputs[?OutputKey==`WorkspaceEndpoint`].OutputValue' \
        --region $REGION \
        --output text
    )
    APS_ENDPOINT="${APS_ENDPOINT}api/v1/remote_write"

    helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
    helm repo add kube-state-metrics https://kubernetes.github.io/kube-state-metrics
    helm repo update

    helm install prometheus prometheus-community/prometheus \
        -n prometheus \
        --set serviceAccounts.server.create=false \
        --set serviceAccounts.server.name=prometheus-server \
        --set server.remoteWrite[0].url=$APS_ENDPOINT \
        --set server.remoteWrite[0].sigv4.region=$REGION \
        --set server.remoteWrite[0].queue_config.max_samples_per_send=1000 \
        --set server.remoteWrite[0].queue_config.max_shards=200 \
        --set server.remoteWrite[0].queue_config.capacity=2500
    ```

=== ":simple-windows: Windows"

    ``` powershell hl_lines="1 2"
    $STACK_NAME="<workspace stack name>"
    $REGION="<region code>"

    $APS_ENDPOINT = aws cloudformation describe-stacks `
        --stack-name $STACK_NAME `
        --query 'Stacks[0].Outputs[?OutputKey==`WorkspaceEndpoint`].OutputValue' `
        --region $REGION `
        --output text
    $APS_ENDPOINT = "$APS_ENDPOINT/api/v1/remote_write"

    helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
    helm repo add kube-state-metrics https://kubernetes.github.io/kube-state-metrics
    helm repo update

    helm install prometheus prometheus-community/prometheus `
        -n prometheus `
        --set serviceAccounts.server.create=false `
        --set serviceAccounts.server.name=prometheus-server `
        --set server.remoteWrite[0].url=$APS_ENDPOINT `
        --set server.remoteWrite[0].sigv4.region=$REGION `
        --set server.remoteWrite[0].queue_config.max_samples_per_send=1000 `
        --set server.remoteWrite[0].queue_config.max_shards=200 `
        --set server.remoteWrite[0].queue_config.capacity=2500
    ```

[AWS Documentation](https://docs.aws.amazon.com/prometheus/latest/userguide/AMP-onboard-ingest-metrics-new-Prometheus.html)