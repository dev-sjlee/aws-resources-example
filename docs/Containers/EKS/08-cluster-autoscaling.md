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
| 1.27       | 1.27.1 |         |
| 1.26       | 1.26.2 | default |
| 1.25       | 1.25.1 |         |
| 1.24       | 1.24.1 |         |
| 1.23       | 1.23   |         |
| 1.22       | 1.22.2 |         |

> Go to [here](https://github.com/kubernetes/autoscaler/releases) and please check the new version of your kubernetes version. (2023-05-25)

[AWS Documentation](https://docs.aws.amazon.com/ko_kr/eks/latest/userguide/autoscaling.html#ca-deploy)

## Using Karpenter (WIP)

### Create the Karpenter namespace

```shell
cat << EOF >> karpenter-namespace.yaml
apiVersion: v1
kind: Namespace
metadata:
  name: karpenter
  labels:
    name: karpenter
EOF

kubectl apply -f karpenter-namespace.yaml
```

### Create the KarpenterNode IAM role

```shell
eksctl create iamidentitymapping \
    --username system:node:{{EC2PrivateDNSName}} \
    --cluster "<cluster name>" \
    --arn "<node IAM role arn>" \
    --group system:bootstrappers \
    --group system:nodes
```

[Karpenter Documentation](https://karpenter.sh/v0.13.2/getting-started/getting-started-with-eksctl/#create-the-karpenternode-iam-role)

### Create the service account for KarPenTerController

```shell
cat << EOF >> karpenter-controller-policy.json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Resource": "*",
      "Action": [
        "ec2:CreateLaunchTemplate",
        "ec2:CreateFleet",
        "ec2:RunInstances",
        "ec2:CreateTags",
        "iam:PassRole",
        "ec2:TerminateInstances",
        "ec2:DeleteLaunchTemplate",
        "ec2:DescribeLaunchTemplates",
        "ec2:DescribeInstances",
        "ec2:DescribeSecurityGroups",
        "ec2:DescribeSubnets",
        "ec2:DescribeInstanceTypes",
        "ec2:DescribeInstanceTypeOfferings",
        "ec2:DescribeAvailabilityZones",
        "ec2:DescribeSpotPriceHistory",
        "ssm:GetParameter",
        "pricing:GetProducts"
      ]
    }
  ]
}
EOF

aws iam create-policy \
    --policy-name <policy name> \
    --policy-document file://karpenter-controller-policy.json

eksctl create iamserviceaccount \
    --cluster <cluster name> \
    --name karpenter \
    --namespace karpenter \
    --role-name <role name> \
    --attach-policy-arn <karpenter controller policy arn> \
    --approve
```
https://karpenter.sh/v0.13.2/getting-started/getting-started-with-eksctl/#create-the-karpentercontroller-iam-role

### Install Karpenter using `helm`

> Please check version in [here](https://karpenter.sh/v0.13.2/getting-started/getting-started-with-eksctl/#environment-variables).

```shell
helm repo add karpenter https://charts.karpenter.sh/
helm repo update

helm upgrade \
    --install \
    --namespace karpenter \
    karpenter karpenter/karpenter \
    --version <karpenter version> \
    --set clusterName=<cluster name> 
    --set clusterEndpoint=<cluster endpoint> \
    --set serviceAccount.create=false \
    --set serviceAccount.name=karpenter \
    --set aws.defaultInstanceProfile=<node IAM role instance profile arn> \
    --wait
```

### Deploy the provisioner

```shell
cat << EOF >> karpenter-provisioner.yaml
apiVersion: karpenter.sh/v1alpha5
kind: Provisioner
metadata:
  name: default
spec:
  limits:
    resources:
      cpu: 100
  provider:
    subnetSelector:
      karpenter.sh/discovery: <cluster name>
    securityGroupSelector:
      karpenter.sh.discovery: <cluster name>
EOF

kubectl apply -f karpenter-provisioner.yaml
```

https://karpenter.sh/v0.13.2/getting-started/getting-started-with-eksctl/#install-karpenter-helm-chart

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