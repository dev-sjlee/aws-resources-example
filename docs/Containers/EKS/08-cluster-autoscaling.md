# Cluster Autoscaling

## Using Cluster Autoscaler(CA)

### Create the service account

=== ":simple-linux: Linux"

    ``` bash hl_lines="1 2 3 4 5"
    POLICY_NAME="<policy name>"
    ROLE_NAME="<role name>"
    CLUSTER_NAME="<cluster name>"
    PROJECT_NAME="<project name>"
    REGION="<region code>"

    curl -LO https://raw.githubusercontent.com/marcus16-kang/aws-resources-example/main/scripts/eks/cluster-autoscaler-policy.json

    POLICY_ARN=$(aws iam create-policy \
        --policy-name $POLICY_NAME \
        --policy-document file://cluster-autoscaler-policy.json \
        --query 'Policy.Arn' \
        --output text \
    #    --tags Key=project,Value=$PROJECT_NAME \ # AWS CLI v2
    )

    eksctl create iamserviceaccount \
        --cluster=$CLUSTER_NAME \
        --namespace=kube-system \
        --name=cluster-autoscaler \
        --role-name=$ROLE_NAME \
        --attach-policy-arn=$POLICY_ARN \
        --region $REGION \
        --override-existing-serviceaccounts \
        --approve
    ```

=== ":simple-windows: Windows"

    ``` powershell hl_lines="1 2 3 4 5"
    $POLICY_NAME="<policy name>"
    $ROLE_NAME="<role name>"
    $CLUSTER_NAME="<cluster name>"
    $PROJECT_NAME="<project name>"
    $REGION="<region code>"

    curl.exe -LO https://raw.githubusercontent.com/marcus16-kang/aws-resources-example/main/scripts/eks/cluster-autoscaler-policy.json

    POLICY_ARN = aws iam create-policy `
        --policy-name $POLICY_NAME `
        --policy-document file://cluster-autoscaler-policy.json `
        --query 'Policy.Arn' `
        --output text `
       --tags Key=project,Value=$PROJECT_NAME `

    eksctl create iamserviceaccount `
        --cluster=$CLUSTER_NAME `
        --namespace=kube-system `
        --name=cluster-autoscaler `
        --role-name=$ROLE_NAME `
        --attach-policy-arn=$POLICY_ARN `
        --region $REGION `
        --override-existing-serviceaccounts `
        --approve
    ```

[AWS Documentation](https://docs.aws.amazon.com/ko_kr/eks/latest/userguide/autoscaling.html#ca-create-policy)

[Autoscaler Documentation](https://github.com/kubernetes/autoscaler/blob/master/cluster-autoscaler/cloudprovider/aws/README.md#iam-policy)

### Deploy the Cluster Autoscaler using `helm`

=== ":simple-linux: Linux"

    ``` bash hl_lines="1 2 3"
    CLUSTER_NAME="<cluster name>"
    IMAGE_TAG="<image tag (ex. 1.25.1)>"
    REGION="<region code>"

    helm repo add autoscaler https://kubernetes.github.io/autoscaler

    helm install cluster-autoscaler autoscaler/cluster-autoscaler \
        --namespace kube-system \
        --set autoDiscovery.clusterName=$CLUSTER_NAME \
        --set awsRegion=$REGION \
        --set cloudProvider=aws \
        --set extraArgs.logtostderr=true \
        --set extraArgs.stderrthreshold=info \
        --set extraArgs.v=4 \
        --set extraArgs.skip-nodes-with-local-storage=false \
        --set extraArgs.expander=least-waste \
        --set extraArgs.balance-similar-node-groups=true \
        --set extraArgs.skip-nodes-with-system-pods=false \
        --set image.tag=v$IMAGE_TAG \
        --set rbac.serviceAccount.create=false \
        --set rbac.serviceAccount.name=cluster-autoscaler
    ```

=== ":simple-windows: Windows"

    ``` bash hl_lines="1 2 3"
    $CLUSTER_NAME="<cluster name>"
    $IMAGE_TAG="<image tag (ex. 1.26.2)>"
    $REGION="<region code>"

    helm repo add autoscaler https://kubernetes.github.io/autoscaler

    helm install cluster-autoscaler autoscaler/cluster-autoscaler `
        --namespace kube-system `
        --set autoDiscovery.clusterName=$CLUSTER_NAME `
        --set awsRegion=$REGION `
        --set cloudProvider=aws `
        --set extraArgs.logtostderr=true `
        --set extraArgs.stderrthreshold=info `
        --set extraArgs.v=4 `
        --set extraArgs.skip-nodes-with-local-storage=false `
        --set extraArgs.expander=least-waste `
        --set extraArgs.balance-similar-node-groups=true `
        --set extraArgs.skip-nodes-with-system-pods=false `
        --set image.tag=v$IMAGE_TAG `
        --set rbac.serviceAccount.create=false `
        --set rbac.serviceAccount.name=cluster-autoscaler
    ```

| Kubernetes | CA     | Default |
|------------|--------|---------|
| 1.27       | 1.27.2 |         |
| 1.26       | 1.26.3 | default |
| 1.25       | 1.25.2 |         |
| 1.24       | 1.24.2 |         |
| 1.23       | 1.23   |         |
| 1.22       | 1.22.2 |         |

> Go to [here](https://github.com/kubernetes/autoscaler/releases) and please check the new version of your kubernetes version. (2023-05-31)

[AWS Documentation](https://docs.aws.amazon.com/ko_kr/eks/latest/userguide/autoscaling.html#ca-deploy)

## Using Karpenter (WIP)

### Add tags to subnets

=== ":simple-linux: Linux"

    ``` bash hl_lines="1 3 4"
    REGION=""

    CLUSTER_NAME=""
    SUBNET_LIST=(
        "subnet-xxxxxxxxxxxxxxxxx"
        "subnet-xxxxxxxxxxxxxxxxx"
    )

    for subnet in "${SUBNET_LIST[@]}"
    do
        aws ec2 create-tags \
            --tags "Key=karpenter.sh/discovery,Value=${CLUSTER_NAME}" \
            --resources ${subnet} \
            --region $REGION \
            --no-cli-pager
    done
    ```

=== ":simple-windows: Windows"

    ``` powershell hl_lines="1 3 4"
    $REGION=""

    $CLUSTER_NAME=""
    $SUBNET_LIST  = @(
        "subnet-xxxxxxxxxxxxxxxxx",
        "subnet-xxxxxxxxxxxxxxxxx"
    )

    foreach ($subnet in $SUBNET_LIST) {
        aws ec2 create-tags `
            --tags "Key=karpenter.sh/discovery,Value=${CLUSTER_NAME}" `
            --resources ${subnet} `
            --region $REGION `
            --no-cli-pager
    }
    ```

### Add tags to security groups

=== ":simple-linux: Linux"

    ``` bash hl_lines="1 3 4"
    REGION=""

    CLUSTER_NAME=""
    SG_LIST=(
        "sg-xxxxxxxxxxxxxxxxx"
        "sg-xxxxxxxxxxxxxxxxx"
    )

    for sg in "${SG_LIST[@]}"
    do
        aws ec2 create-tags \
            --tags "Key=karpenter.sh/discovery,Value=${CLUSTER_NAME}" \
            --resources ${sg} \
            --region $REGION \
            --no-cli-pager
    done
    ```

=== ":simple-windows: Windows"

    ``` powershell hl_lines="1 3 4"
    $REGION=""

    $CLUSTER_NAME=""
    $SG_LIST  = @(
        "sg-xxxxxxxxxxxxxxxxx",
        "sg-xxxxxxxxxxxxxxxxx"
    )

    foreach ($sg in $SG_LIST) {
        aws ec2 create-tags `
            --tags "Key=karpenter.sh/discovery,Value=${CLUSTER_NAME}" `
            --resources ${sg} `
            --region $REGION `
            --no-cli-pager
    }
    ```

### Create the Karpenter Controller policy

=== ":simple-linux: Linux"

    ``` bash hl_lines="1 2 3 6 9 10 13"
    STACK_NAME=""
    PROJECT_NAME=""
    REGION=""

    ### Policy Configuration
    PolicyName=""     # [REQUIRED] The name of service account's IAM policy.

    ### EKS Configuration
    ClusterName=""    # [REQUIRED] The name of EKS cluster.
    NodeRoleName=""   # [REQUIRED] The name of EKS node's role.

    ### SQS Configuration
    QueueName=""      # [REQUIRED] The name of SQS queue for NTH.

    curl -LO https://raw.githubusercontent.com/marcus16-kang/aws-resources-example/main/scripts/eks/karpenter-controller-policy.yaml

    aws cloudformation deploy \
        --template-file ./karpenter-controller-policy.yaml \
        --stack-name $STACK_NAME \
        --parameter-overrides \
            PolicyName=$PolicyName \
            ClusterName=$ClusterName \
            NodeRoleName=$NodeRoleName \
            QueueName=$QueueName \
        --tags project=$PROJECT_NAME \
        --capabilities CAPABILITY_NAMED_IAM \
        --disable-rollback \
        --region $REGION
    ```

=== ":simple-windows: Windows"

    ``` powershell hl_lines="1 2 3 6 9 10 13"
    $STACK_NAME=""
    $PROJECT_NAME=""
    $REGION=""

    ### Policy Configuration
    $PolicyName=""    # [REQUIRED] The name of service account's IAM policy.

    ### EKS Configuration
    $ClusterName=""   # [REQUIRED] The name of EKS cluster.
    $NodeRoleName=""  # [REQUIRED] The name of EKS node's role.

    ### SQS Configuration
    $QueueName=""      # [REQUIRED] The name of SQS queue for NTH

    curl.exe -LO https://raw.githubusercontent.com/marcus16-kang/aws-resources-example/main/scripts/eks/karpenter-controller-policy.yaml

    aws cloudformation deploy `
    --template-file ./karpenter-controller-policy.yaml `
    --stack-name $STACK_NAME `
    --parameter-overrides `
        PolicyName=$PolicyName `
        ClusterName=$ClusterName `
        NodeRoleName=$NodeRoleName `
        QueueName=$QueueName `
    --tags project=$PROJECT_NAME `
    --capabilities CAPABILITY_NAMED_IAM `
    --disable-rollback `
    --region $REGION
    ```

[Karpenter Documentation](https://karpenter.sh/preview/getting-started/migrating-from-cas/#create-iam-roles)

### Create the service account

=== ":simple-linux: Linux"

    ``` bash hl_lines="1 2 3 4 5"
    POLICY_NAME="<policy name>"
    ROLE_NAME="<role name>"
    CLUSTER_NAME="<cluster name>"
    PROJECT_NAME="<project name>"
    REGION="<region code>"

    POLICY_ARN=$(aws iam list-policies \
        --query "Policies[?PolicyName=='$POLICY_NAME'].Arn" --output text)

    eksctl create iamserviceaccount \
        --cluster=$CLUSTER_NAME \
        --namespace=karpenter \
        --name=karpenter \
        --role-name=$ROLE_NAME \
        --attach-policy-arn=$POLICY_ARN \
        --region $REGION \
        --approve
    ```

=== ":simple-windows: Windows"

    ``` powershell hl_lines="1 2 3 4 5"
    $POLICY_NAME="<policy name>"
    $ROLE_NAME="<role name>"
    $CLUSTER_NAME="<cluster name>"
    $PROJECT_NAME="<project name>"
    $REGION="<region code>"

    $POLICY_ARN = aws iam list-policies `
        --query "Policies[?PolicyName=='$POLICY_NAME'].Arn" --output text

    eksctl create iamserviceaccount `
        --cluster=$CLUSTER_NAME `
        --namespace=karpenter `
        --name=karpenter `
        --role-name=$ROLE_NAME `
        --attach-policy-arn=$POLICY_ARN `
        --region $REGION `
        --approve
    ```

!!! note

    If you use CMK in SQS queue, you should add below policy.

    ``` json title="KMS Key Policy" hl_lines="6"
    {
        "Sid": "AllowConsumersToReceiveFromTheQueue - Karpenter",
        "Effect": "Allow",
        "Principal": {
            "AWS": [
                "<KARPENTER SERVICE ACCOUNT ROLE ARN>"
            ]
        },
        "Action": [
            "kms:Decrypt"
        ],
        "Resource": "*"
    }
    ```

### Install Karpenter using `helm`

=== ":simple-linux: Linux"

    ``` bash hl_lines="1 2 3"
    CLUSTER_NAME=""
    NODEGROUP_ROLE_NAME=""
    QUEUE_NAME=""

    helm repo add karpenter https://charts.karpenter.sh/
    helm repo update

    helm install karpenter oci://public.ecr.aws/karpenter/karpenter \
        --create-namespace \
        --namespace karpenter \
        --version v0.27.5 \
        --set serviceAccount.create=false \
        --set serviceAccount.name=karpenter \
        --set settings.aws.clusterName=$CLUSTER_NAME \
        --set settings.aws.defaultInstanceProfile=$NODEGROUP_ROLE_NAME \
        --set settings.aws.interruptionQueueName=$QUEUE_NAME
    ```

=== ":simple-windows: Windows"

    ``` bash hl_lines="1 2 3"
    $CLUSTER_NAME=""
    $NODEGROUP_ROLE_NAME=""
    $QUEUE_NAME=""

    helm repo add karpenter https://charts.karpenter.sh/
    helm repo update

    helm install karpenter oci://public.ecr.aws/karpenter/karpenter `
        --create-namespace `
        --namespace karpenter `
        --version v0.27.5 `
        --set serviceAccount.create=false `
        --set serviceAccount.name=karpenter `
        --set settings.aws.clusterName=$CLUSTER_NAME `
        --set settings.aws.defaultInstanceProfile=$NODEGROUP_ROLE_NAME `
        --set settings.aws.interruptionQueueName=$QUEUE_NAME
    ```

> Go to [here](https://github.com/aws/karpenter/releases) and please check the new version. (2023-06-02)

### Create a node template

``` yaml title="node-template.yaml" linenums="1"
apiVersion: karpenter.k8s.aws/v1alpha1
kind: AWSNodeTemplate
metadata:
  name: <node template name>
spec:
  subnetSelector: { ... }        # required, discovers tagged subnets to attach to instances
  securityGroupSelector: { ... } # required, discovers tagged security groups to attach to instances
  instanceProfile: "..."         # optional, overrides the node's identity from global settings
  amiFamily: "..."               # optional, resolves a default ami and userdata
  amiSelector: { ... }           # optional, discovers tagged amis to override the amiFamily's default
  userData: "..."                # optional, overrides autogenerated userdata with a merge semantic
  tags: { ... }                  # optional, propagates tags to underlying EC2 resources
  metadataOptions: { ... }       # optional, configures IMDS for the instance
  blockDeviceMappings: [ ... ]   # optional, configures storage devices for the instance
  detailedMonitoring: "..."      # optional, configures detailed monitoring for the instance
```

??? note "Amazon Linux 2 Example"

    ``` yaml title="node-template.yaml" linenums="1"
    apiVersion: karpenter.k8s.aws/v1alpha1
    kind: AWSNodeTemplate
    metadata:
      name: <node-template-name>
    spec:
      subnetSelector:
        aws-ids: "subnet-xxxxxxxxxxxxxxxxx,subnet-xxxxxxxxxxxxxxxxx"
      securityGroupSelector:
        aws-ids: "sg-xxxxxxxxxxxxxxxxx,sg-xxxxxxxxxxxxxxxxx"
      instanceProfile: <instance-profile-name>
      amiFamily: AL2
      tags:
        Name: <instance, volume, lt name>
        project: <project name>
      metadataOptions:
        httpEndpoint: enabled
        httpProtocolIPv6: disabled
        httpPutResponseHopLimit: 2
        httpTokens: required
      blockDeviceMappings:
        - deviceName: /dev/xvda
          ebs:
            volumeSize: 50Gi
            volumeType: gp3
            encrypted: true
    ```

??? note "Bottlerocket Example"

    ``` yaml title="node-template.yaml" linenums="1"
    apiVersion: karpenter.k8s.aws/v1alpha1
    kind: AWSNodeTemplate
    metadata:
      name: <node-template-name>
    spec:
      subnetSelector:
        aws-ids: "subnet-xxxxxxxxxxxxxxxxx,subnet-xxxxxxxxxxxxxxxxx"
      securityGroupSelector:
        aws-ids: "sg-xxxxxxxxxxxxxxxxx,sg-xxxxxxxxxxxxxxxxx"
      instanceProfile: <instance-profile-name>
      amiFamily: Bottlerocket
      tags:
        Name: <instance, volume, lt name>
        project: <project name>
      metadataOptions:
        httpEndpoint: enabled
        httpProtocolIPv6: disabled
        httpPutResponseHopLimit: 2
        httpTokens: required
      blockDeviceMappings:
        # Root device
        - deviceName: /dev/xvda
          ebs:
            volumeSize: 4Gi
            volumeType: gp3
            encrypted: true
        # Data device: Container resources such as images and logs
        - deviceName: /dev/xvdb
          ebs:
            volumeSize: 50Gi
            volumeType: gp3
            encrypted: true
    ```

[Karpenter Documentation](https://karpenter.sh/v0.27.5/concepts/node-templates/)

### Create a provisioner

``` yaml title="provisioner" linenums="1"

```

[Karpenter Documentation](https://karpenter.sh/v0.27.5/concepts/provisioners/)

## Deploy Overprovisioning Pod

``` yaml title="overprovisioning.yaml"
---
apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata:
  name: overprovisioning
value: -1
globalDefault: false
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: overprovisioning
  namespace: kube-system
spec:
  replicas: 3
  selector:
    matchLabels:
      run: overprovisioning
  template:
    metadata:
      labels:
        run: overprovisioning
    spec:
      priorityClassName: overprovisioning
      containers:
      - name: reserve-resources
        image: k8s.gcr.io/pause
        resources:
          requests:
            cpu: 820m
            memory: 2000Mi
      # tolerations:
      #   - key: "key1"         # taint key
      #     value: "value1"     # taint value
      #     operator: "Equal"
      #     effect: "NoSchedule"
      # nodeSelector:
      #   key: value            # node label key and value
```