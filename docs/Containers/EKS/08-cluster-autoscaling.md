# Cluster Autoscaling

## Using Cluster Autoscaler(CA)

### Create the service account

=== "JSON file"
    ``` json hl_lines="14"
    {
      "Version": "2012-10-17",
      "Statement": [
        {
          "Effect": "Allow",
          "Action": [
            "autoscaling:DescribeAutoScalingGroups",
            "autoscaling:DescribeAutoScalingInstances",
            "autoscaling:DescribeLaunchConfigurations",
            "autoscaling:DescribeScalingActivities",
            "autoscaling:DescribeTags",
            "ec2:DescribeInstanceTypes",
            "ec2:DescribeLaunchTemplateVersions"
          ],
          "Resource": ["*"]
        },
        {
          "Effect": "Allow",
          "Action": [
            "autoscaling:SetDesiredCapacity",
            "autoscaling:TerminateInstanceInAutoScalingGroup",
            "ec2:DescribeImages",
            "ec2:GetInstanceTypesFromInstanceRequirements",
            "eks:DescribeNodegroup"
          ],
          "Resource": ["*"]
        }
      ]
    }
    ```

=== "Using command" 
    ``` shell hl_lines="15 36 40 43 44"
    cat << EOF >> cluster-autoscaler-policy.json
    {
      "Version": "2012-10-17",
      "Statement": [
        {
          "Effect": "Allow",
          "Action": [
            "autoscaling:DescribeAutoScalingGroups",
            "autoscaling:DescribeAutoScalingInstances",
            "autoscaling:DescribeLaunchConfigurations",
            "autoscaling:DescribeScalingActivities",
            "autoscaling:DescribeTags",
            "ec2:DescribeInstanceTypes",
            "ec2:DescribeLaunchTemplateVersions"
          ],
          "Resource": ["*"]
        },
        {
          "Effect": "Allow",
          "Action": [
            "autoscaling:SetDesiredCapacity",
            "autoscaling:TerminateInstanceInAutoScalingGroup",
            "ec2:DescribeImages",
            "ec2:GetInstanceTypesFromInstanceRequirements",
            "eks:DescribeNodegroup"
          ],
          "Resource": ["*"]
        }
      ]
    }
    EOF

    aws iam create-policy \
        --policy-name <policy name> \
        --policy-document file://cluster-autoscaler-policy.json

    eksctl create iamserviceaccount \
        --cluster=<cluster name> \
        --namespace=kube-system \
        --name=cluster-autoscaler \
        --role-name=<role name> \
        --attach-policy-arn=arn:aws:iam::<account id>:policy/<policy name> \
        --override-existing-serviceaccounts \
        --approve
    ```

[AWS Documentation](https://docs.aws.amazon.com/ko_kr/eks/latest/userguide/autoscaling.html#ca-create-policy)
[Autoscaler Documentation](https://github.com/kubernetes/autoscaler/blob/master/cluster-autoscaler/cloudprovider/aws/README.md#iam-policy)

### Deploy the Cluster Autoscaler

``` shell
curl -o cluster-autoscaler-autodiscover.yaml https://raw.githubusercontent.com/kubernetes/autoscaler/master/cluster-autoscaler/cloudprovider/aws/examples/cluster-autoscaler-autodiscover.yaml

kubectl apply -f cluster-autoscaler-autodiscover.yaml

kubectl patch deployment cluster-autoscaler \
    -n kube-system \
    -p '{"spec":{"template":{"metadata":{"annotations":{"cluster-autoscaler.kubernetes.io/safe-to-evict": "false"}}}}}'

kubectl -n kube-system edit deployment.apps/cluster-autoscaler
```

> Please add `--balance-similar-node-groups` and `--skip-nodes-with-system-pods=false` at `spec.template.spec.containers.command`, and change to your cluster name like this:

``` yaml
    spec:
      containers:
      - command
        - ./cluster-autoscaler
        - --v=4
        - --stderrthreshold=info
        - --cloud-provider=aws
        - --skip-nodes-with-local-storage=false
        - --expander=least-waste
        - --node-group-auto-discovery=asg:tag=k8s.io/cluster-autoscaler/enabled,k8s.io/cluster-autoscaler/<YOUR CLUSTER NAME>
        - --balance-similar-node-groups
        - --skip-nodes-with-system-pods=false
```

> Go to [here](https://github.com/kubernetes/autoscaler/releases) and please check the new version of your kubernetes version.

```shell
kubectl set image deployment cluster-autoscaler \
    -n kube-system \
    cluster-autoscaler=k8s.gcr.io/autoscaling/cluster-autoscaler:v<1.22.n>
```

> for example: `cluster-autoscaler=k8s.gcr.io/autoscaling/cluster-autoscaler:v1.22.3`

<details>
<summary>cluster-autoscaler-autodiscover.yaml</summary>
<div markdown="1">

```yaml
---
apiVersion: v1
kind: ServiceAccount
metadata:
  labels:
    k8s-addon: cluster-autoscaler.addons.k8s.io
    k8s-app: cluster-autoscaler
  name: cluster-autoscaler
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: cluster-autoscaler
  labels:
    k8s-addon: cluster-autoscaler.addons.k8s.io
    k8s-app: cluster-autoscaler
rules:
  - apiGroups: [""]
    resources: ["events", "endpoints"]
    verbs: ["create", "patch"]
  - apiGroups: [""]
    resources: ["pods/eviction"]
    verbs: ["create"]
  - apiGroups: [""]
    resources: ["pods/status"]
    verbs: ["update"]
  - apiGroups: [""]
    resources: ["endpoints"]
    resourceNames: ["cluster-autoscaler"]
    verbs: ["get", "update"]
  - apiGroups: [""]
    resources: ["nodes"]
    verbs: ["watch", "list", "get", "update"]
  - apiGroups: [""]
    resources:
      - "namespaces"
      - "pods"
      - "services"
      - "replicationcontrollers"
      - "persistentvolumeclaims"
      - "persistentvolumes"
    verbs: ["watch", "list", "get"]
  - apiGroups: ["extensions"]
    resources: ["replicasets", "daemonsets"]
    verbs: ["watch", "list", "get"]
  - apiGroups: ["policy"]
    resources: ["poddisruptionbudgets"]
    verbs: ["watch", "list"]
  - apiGroups: ["apps"]
    resources: ["statefulsets", "replicasets", "daemonsets"]
    verbs: ["watch", "list", "get"]
  - apiGroups: ["storage.k8s.io"]
    resources: ["storageclasses", "csinodes", "csidrivers", "csistoragecapacities"]
    verbs: ["watch", "list", "get"]
  - apiGroups: ["batch", "extensions"]
    resources: ["jobs"]
    verbs: ["get", "list", "watch", "patch"]
  - apiGroups: ["coordination.k8s.io"]
    resources: ["leases"]
    verbs: ["create"]
  - apiGroups: ["coordination.k8s.io"]
    resourceNames: ["cluster-autoscaler"]
    resources: ["leases"]
    verbs: ["get", "update"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: cluster-autoscaler
  namespace: kube-system
  labels:
    k8s-addon: cluster-autoscaler.addons.k8s.io
    k8s-app: cluster-autoscaler
rules:
  - apiGroups: [""]
    resources: ["configmaps"]
    verbs: ["create","list","watch"]
  - apiGroups: [""]
    resources: ["configmaps"]
    resourceNames: ["cluster-autoscaler-status", "cluster-autoscaler-priority-expander"]
    verbs: ["delete", "get", "update", "watch"]

---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: cluster-autoscaler
  labels:
    k8s-addon: cluster-autoscaler.addons.k8s.io
    k8s-app: cluster-autoscaler
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-autoscaler
subjects:
  - kind: ServiceAccount
    name: cluster-autoscaler
    namespace: kube-system

---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: cluster-autoscaler
  namespace: kube-system
  labels:
    k8s-addon: cluster-autoscaler.addons.k8s.io
    k8s-app: cluster-autoscaler
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: cluster-autoscaler
subjects:
  - kind: ServiceAccount
    name: cluster-autoscaler
    namespace: kube-system

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: cluster-autoscaler
  namespace: kube-system
  labels:
    app: cluster-autoscaler
spec:
  replicas: 1
  selector:
    matchLabels:
      app: cluster-autoscaler
  template:
    metadata:
      labels:
        app: cluster-autoscaler
      annotations:
        prometheus.io/scrape: 'true'
        prometheus.io/port: '8085'
    spec:
      priorityClassName: system-cluster-critical
      securityContext:
        runAsNonRoot: true
        runAsUser: 65534
        fsGroup: 65534
      serviceAccountName: cluster-autoscaler
      containers:
        - image: k8s.gcr.io/autoscaling/cluster-autoscaler:v1.22.2
          name: cluster-autoscaler
          resources:
            limits:
              cpu: 100m
              memory: 600Mi
            requests:
              cpu: 100m
              memory: 600Mi
          command:
            - ./cluster-autoscaler
            - --v=4
            - --stderrthreshold=info
            - --cloud-provider=aws
            - --skip-nodes-with-local-storage=false
            - --expander=least-waste
            - --node-group-auto-discovery=asg:tag=k8s.io/cluster-autoscaler/enabled,k8s.io/cluster-autoscaler/<YOUR CLUSTER NAME>
          volumeMounts:
            - name: ssl-certs
              mountPath: /etc/ssl/certs/ca-certificates.crt #/etc/ssl/certs/ca-bundle.crt for Amazon Linux Worker Nodes
              readOnly: true
          imagePullPolicy: "Always"
      volumes:
        - name: ssl-certs
          hostPath:
            path: "/etc/ssl/certs/ca-bundle.crt"
```

</div>
</details>

[AWS Documentation](https://docs.aws.amazon.com/ko_kr/eks/latest/userguide/autoscaling.html#ca-deploy)

## Using Karpenter

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

https://karpenter.sh/v0.13.2/getting-started/getting-started-with-eksctl/#create-the-karpenternode-iam-role

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
apiVersion: scheduling.k8s.io/v1beta1
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
```