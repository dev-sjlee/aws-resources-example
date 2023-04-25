# Using EBS

## Create service account using `eksctl`

=== ":simple-linux: Linux"

    ``` bash hl_lines="1 2 3"
    CLUSTER_NAME="<cluster name>"
    ROLE_NAME="<role name>"
    REGION="<region code>"

    eksctl create iamserviceaccount \
        --name ebs-csi-controller-sa \
        --namespace kube-system \
        --cluster $CLUSTER_NAME \
        --attach-policy-arn arn:aws:iam::aws:policy/service-role/AmazonEBSCSIDriverPolicy \
        --role-name $ROLE_NAME \
        --role-only \
        --region $REGION \
        --approve
    ```

=== ":simple-windows: Windows"

    ``` powershell hl_lines="1 2 3"
    $CLUSTER_NAME="<cluster name>"
    $ROLE_NAME="<role name>"
    $REGION="<region code>"

    eksctl create iamserviceaccount `
        --name ebs-csi-controller-sa `
        --namespace kube-system `
        --cluster $CLUSTER_NAME `
        --attach-policy-arn arn:aws:iam::aws:policy/service-role/AmazonEBSCSIDriverPolicy `
        --role-name $ROLE_NAME `
        --role-only `
        --region $REGION `
        --approve
    ```

[AWS Documentation](https://docs.aws.amazon.com/ko_kr/eks/latest/userguide/csi-iam-role.html)

## Install EBS CSI Driver

=== ":simple-linux: Linux"

    ``` bash hl_lines="1 2 3"
    CLUSTER_NAME="<cluster name>"
    ROLE_NAME="<role name>"
    REGION="<region code>"

    ROLE_ARN=$(aws iam get-role --role-name $ROLE_NAME --region $REGION --query 'Role.Arn' --output text)

    eksctl create addon \
        --name aws-ebs-csi-driver \
        --cluster $CLUSTER_NAME \
        --service-account-role-arn $ROLE_ARN \
        --region $REGION \
        --force
    ```

=== ":simple-windows: Windows"

    ``` powershell hl_lines="1 2 3"
    $CLUSTER_NAME="<cluster name>"
    $ROLE_NAME="<role name>"
    $REGION="<region code>"

    $ROLE_ARN = aws iam get-role --role-name $ROLE_NAME --region $REGION --query 'Role.Arn' --output text

    eksctl create addon `
        --name aws-ebs-csi-driver `
        --cluster $CLUSTER_NAME `
        --service-account-role-arn $ROLE_ARN `
        --region $REGION `
        --force
    ```

[AWS Documentation](https://docs.aws.amazon.com/ko_kr/eks/latest/userguide/efs-csi.html#efs-install-driver)

## Use EBS File System

### Static Provisioning

!!! note

    You can see examples in [HERE](https://github.com/kubernetes-sigs/aws-ebs-csi-driver/tree/master/examples/kubernetes/static-provisioning).

``` yaml linenums="1" title="persistent-volume.yaml"
apiVersion: v1
kind: PersistentVolume
metadata:
  name: ebs-pv
spec:
  accessModes:
  - ReadWriteOnce
  capacity:
    storage: 5Gi
  csi:
    driver: ebs.csi.aws.com
    fsType: ext4
    volumeHandle: vol-XXXXXXXXXXXXXXXXX
  nodeAffinity:
    required:
      nodeSelectorTerms:
        - matchExpressions:
            - key: topology.ebs.csi.aws.com/zone
              operator: In
              values:
                - us-east-2a
```

``` yaml linenums="1" title="persistent-volume-claim.yaml"
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: ebs-claim
  namespace: default
spec:
  storageClassName: "" # Empty string must be explicitly set otherwise default StorageClass will be set
  volumeName: ebs-pv
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi
```

``` yaml linenums="1" title="pod.yaml"
apiVersion: v1
kind: Pod
metadata:
  name: app
  namespace: default
spec:
  containers:
  - name: app
    image: centos
    command: ["/bin/sh"]
    args: ["-c", "while true; do echo $(date -u) >> /data/out.txt; sleep 5; done"]
    volumeMounts:
    - name: persistent-storage
      mountPath: /data
  volumes:
  - name: persistent-storage
    persistentVolumeClaim:
      claimName: ebs-claim
```

### Dynamic Provisioning

!!! note

    You can see examples in [HERE](https://github.com/kubernetes-sigs/aws-ebs-csi-driver/tree/master/examples/kubernetes/dynamic-provisioning).

``` yaml linenums="1" title="storage-class.yaml"
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: ebs-sc
provisioner: ebs.csi.aws.com
volumeBindingMode: WaitForFirstConsumer
```

``` yaml linenums="1" title="persistent-volume-claim.yaml"
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: ebs-claim
  namespace: default
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: ebs-sc
  resources:
    requests:
      storage: 4Gi
```

``` yaml linenums="1" title="pod.yaml"
apiVersion: v1
kind: Pod
metadata:
  name: app
  namespace: default
spec:
  containers:
  - name: app
    image: centos
    command: ["/bin/sh"]
    args: ["-c", "while true; do echo $(date -u) >> /data/out.txt; sleep 5; done"]
    volumeMounts:
    - name: persistent-storage
      mountPath: /data
  volumes:
  - name: persistent-storage
    persistentVolumeClaim:
      claimName: ebs-claim
```