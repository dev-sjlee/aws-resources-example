# Using EFS

## Create service account using IAM role

=== ":simple-linux: Linux"

    ``` bash hl_lines="1 2 3 4 5"
    CLUSTSER_NAME="<cluster name>"
    POLICY_NAME="<policy name>"
    ROLE_NAME="<role name>"
    PROJECT_NAME="<project name>"
    REGION="<region code>"

    curl -Lo efs-csi-driver-policy.json https://raw.githubusercontent.com/kubernetes-sigs/aws-efs-csi-driver/master/docs/iam-policy-example.json

    POLICY_ARN=$(aws iam create-policy \
        --policy-name $POLICY_NAME \
        --policy-document file://efs-csi-driver-policy.json \
        # --tags Key=project,Value=$PROJECT_NAME \  # AWS CLI v2
    | jq -r '.Policy.Arn')

    eksctl create iamserviceaccount \
        --cluster $CLUSTER_NAME \
        --namespace=kube-system \
        --name=efs-csi-controller-sa \
        --attach-policy-arn $POLICY_ARN \
        --role-name $ROLE_NAME \
        --tags project=$PROJECT_NAME \
        --region $REGION \
        --override-existing-serviceaccounts \
        --approve
    ```

=== ":simple-windows: Windows"

    ``` powershell hl_lines="1 2 3 4 5"
    $CLUSTSER_NAME="<cluster name>"
    $POLICY_NAME="<policy name>"
    $ROLE_NAME="<role name>"
    $PROJECT_NAME="<project name>"
    $REGION="<region code>"

    curl.exe -Lo efs-csi-driver-policy.json https://raw.githubusercontent.com/kubernetes-sigs/aws-efs-csi-driver/master/docs/iam-policy-example.json

    $POLICY_ARN = aws iam create-policy `
        --policy-name $POLICY_NAME `
        --policy-document file://efs-csi-driver-policy.json `
        --tags Key=project,Value=$PROJECT_NAME `
        --query 'Policy.Arn' `
        --output text

    eksctl create iamserviceaccount `
        --cluster $CLUSTER_NAME `
        --namespace=kube-system `
        --name=efs-csi-controller-sa `
        --attach-policy-arn $POLICY_ARN `
        --role-name $ROLE_NAME `
        --tags project=$PROJECT_NAME `
        --region $REGION `
        --override-existing-serviceaccounts `
        --approve
    ```

??? note "efs-csi-driver-policy.json"

    ``` json
    {
      "Version": "2012-10-17",
      "Statement": [
        {
          "Effect": "Allow",
          "Action": [
            "elasticfilesystem:DescribeAccessPoints",
            "elasticfilesystem:DescribeFileSystems",
            "elasticfilesystem:DescribeMountTargets",
            "ec2:DescribeAvailabilityZones"
          ],
          "Resource": "*"
        },
        {
          "Effect": "Allow",
          "Action": [
            "elasticfilesystem:CreateAccessPoint"
          ],
          "Resource": "*",
          "Condition": {
            "StringLike": {
              "aws:RequestTag/efs.csi.aws.com/cluster": "true"
            }
          }
        },
        {
          "Effect": "Allow",
          "Action": "elasticfilesystem:DeleteAccessPoint",
          "Resource": "*",
          "Condition": {
            "StringEquals": {
              "aws:ResourceTag/efs.csi.aws.com/cluster": "true"
            }
          }
        }
      ]
    }
    ```

[AWS Documentation](https://docs.aws.amazon.com/ko_kr/eks/latest/userguide/efs-csi.html#efs-create-iam-resources)

## Install EFS Driver

=== ":simple-linux: Linux"

    ``` bash hl_lines="1"
    REGION="<region code>"

    helm repo add aws-efs-csi-driver https://kubernetes-sigs.github.io/aws-efs-csi-driver/
    helm repo update

    helm upgrade -i aws-efs-csi-driver aws-efs-csi-driver/aws-efs-csi-driver \
        --namespace kube-system \
        --set image.repository=602401143452.dkr.ecr.$REGION.amazonaws.com/eks/aws-efs-csi-driver \
        --set controller.serviceAccount.create=false \
        --set controller.serviceAccount.name=efs-csi-controller-sa
    ```

=== ":simple-windows: Windows"

    ``` powershell hl_lines="1"
    $REGION="<region code>"

    helm repo add aws-efs-csi-driver https://kubernetes-sigs.github.io/aws-efs-csi-driver/
    helm repo update

    helm upgrade -i aws-efs-csi-driver aws-efs-csi-driver/aws-efs-csi-driver `
        --namespace kube-system `
        --set image.repository=602401143452.dkr.ecr.$REGION.amazonaws.com/eks/aws-efs-csi-driver `
        --set controller.serviceAccount.create=false `
        --set controller.serviceAccount.name=efs-csi-controller-sa
    ```

!!! note

    You should check registry account id from [here](https://docs.aws.amazon.com/ko_kr/eks/latest/userguide/add-ons-images.html).

[AWS Documentation](https://docs.aws.amazon.com/ko_kr/eks/latest/userguide/efs-csi.html#efs-install-driver)

## Use EFS File System

### Static Provisioning

!!! note

    You can see examples in [HERE](https://github.com/kubernetes-sigs/aws-efs-csi-driver/tree/master/examples/kubernetes/static_provisioning).

``` yaml title="persistent-volume.yaml"
apiVersion: v1
kind: PersistentVolume
metadata:
  name: efs-pv
spec:
  capacity:
    storage: 5Gi
  volumeMode: Filesystem
  accessModes:
    - ReadWriteMany
  storageClassName: ""
  persistentVolumeReclaimPolicy: Retain   # `Retain` or `Delete`
  csi:
    driver: efs.csi.aws.com
    volumeHandle: fs-XXXXXXXXXXXXXXXXX
```

``` yaml title="persistent-volume-claim.yaml"
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: efs-claim
  namespace: default
spec:
  accessModes:
    - ReadWriteMany
  storageClassName: ""
  resources:
    requests:
      storage: 5Gi
```

``` yaml title="pod.yaml"
apiVersion: apps/v1
kind: Pod
metadata:
  name: nginx
  namespace: default
  labels:
    app: nginx
spec:
  containers:
    - name: nginx
      image: public.ecr.aws/docker/library/nginx:1.24.0-alpine3.17-slim
      volumeMounts:
        - name: persistent-volume
          mountPath: /usr/share/nginx/html
  volumes:
    - name: persistent-volume
      persistentVolumeClaim:
        claimName: efs-claim
```

### Dynamic Provisioning

!!! note

    You can see examples in [HERE](https://github.com/kubernetes-sigs/aws-efs-csi-driver/tree/master/examples/kubernetes/dynamic_provisioning).

``` yaml title="storage-class.yaml"
kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
  name: efs-sc
provisioner: efs.csi.aws.com
parameters:
  provisioningMode: efs-ap
  fileSystemId: fs-XXXXXXXXXXXXXXXXX
  directoryPerms: "700"
  uid: "101"
  gid: "101"
  basePath: "/nginx" # optional
```

``` yaml title="persistent-volume-claim.yaml"
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: efs-claim
  namespace: default
spec:
  accessModes:
    - ReadWriteMany
  storageClassName: efs
  resources:
    requests:
      storage: 5Gi
```

``` yaml title="deployment.yaml"
apiVersion: apps/v1
kind: Pod
metadata:
  name: nginx
  namespace: default
  labels:
    app: nginx
spec:
  containers:
    - name: nginx
      image: public.ecr.aws/docker/library/nginx:1.24.0-alpine3.17-slim
      volumeMounts:
        - name: persistent-volume
          mountPath: /usr/share/nginx/html
  volumes:
    - name: persistent-volume
      persistentVolumeClaim:
        claimName: efs-claim
```