---
title: Using FSx for Lustre
description: Install and use Amazon FSx for Lustre
---

# Using FSx for Lustre

## Create service account using IAM role

=== ":simple-linux: Linux"

    ``` bash hl_lines="1 2 3 4 5"
    CLUSTER_NAME="<cluster name>"
    ROLE_NAME="<role name>"
    PROJECT_NAME="<project name>"
    REGION="<region code>"

    eksctl create iamserviceaccount \
        --cluster $CLUSTER_NAME \
        --namespace=kube-system \
        --name=fsx-csi-controller-sa \
        --attach-policy-arn=arn:aws:iam::aws:policy/AmazonFSxFullAccess \
        --role-name $ROLE_NAME \
        --tags project=$PROJECT_NAME \
        --region $REGION \
        --override-existing-serviceaccounts \
        --approve
    ```

=== ":simple-windows: Windows"

    ``` powershell hl_lines="1 2 3 4 5"
    $CLUSTER_NAME="<cluster name>"
    $ROLE_NAME="<role name>"
    $PROJECT_NAME="<project name>"
    $REGION="<region code>"

    eksctl create iamserviceaccount `
        --cluster $CLUSTER_NAME `
        --namespace=kube-system `
        --name=fsx-csi-controller-sa `
        --attach-policy-arn=arn:aws:iam::aws:policy/AmazonFSxFullAccess `
        --role-name $ROLE_NAME `
        --tags project=$PROJECT_NAME `
        --region $REGION `
        --override-existing-serviceaccounts `
        --approve
    ```

## Install FSx Driver

=== ":simple-linux: Linux"

    ``` bash
    helm repo add aws-fsx-csi-driver https://kubernetes-sigs.github.io/aws-fsx-csi-driver/
    helm repo update
    helm install aws-fsx-csi-driver aws-fsx-csi-driver/aws-fsx-csi-driver \
        --namespace kube-system \
        --set controller.serviceAccount.create=false \
        --set node.tolerateAllTaints=true
    ```

=== ":simple-windows: Windows"

    ``` powershell
    helm repo add aws-fsx-csi-driver https://kubernetes-sigs.github.io/aws-fsx-csi-driver/
    helm repo update
    helm install aws-fsx-csi-driver aws-fsx-csi-driver/aws-fsx-csi-driver `
        --namespace kube-system `
        --set controller.serviceAccount.create=false `
        --set node.tolerateAllTaints=true
    ```

## Use FSx for Lustre File System

### Static Provisioning

!!! note

    You can see examples in [HERE](https://github.com/kubernetes-sigs/aws-fsx-csi-driver/tree/master/examples/kubernetes/static_provisioning).

``` yaml linenums="1" title="persistent-volume.yaml"
apiVersion: v1
kind: PersistentVolume
metadata:
  name: fsx-pv
spec:
  capacity:
    storage: 1200Gi
  volumeMode: Filesystem
  accessModes:
    - ReadWriteMany
  mountOptions:
    - flock
  persistentVolumeReclaimPolicy: Retain
  csi:
    driver: fsx.csi.aws.com
    volumeHandle: fs-XXXXXXXXXXXXXXXXX
    volumeAttributes:
      dnsname: fs-XXXXXXXXXXXXXXXXX.fsx.us-east-1.amazonaws.com
      mountname: fsx
```

!!! note

    Replace `volumeHandle` with `FileSystemId`, `dnsname` with `DNSName` and `mountname` with `MountName`. You can get both `FileSystemId`, `DNSName` and `MountName` using AWS CLI:

    ``` shell
    aws fsx describe-file-systems
    ```

``` yaml linenums="1" title="persistent-volume-claim.yaml"
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: fsx-claim
  namespace: default
spec:
  accessModes:
    - ReadWriteMany
  storageClassName: ""
  resources:
    requests:
      storage: 1200Gi
  volumeName: fsx-pv
```

``` yaml linenums="1" title="pod.yaml"
apiVersion: v1
kind: Pod
metadata:
  name: fsx-app
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
      claimName: fsx-claim
```

### Dynamic Provisioning

!!! note

    You can see examples in [HERE](https://github.com/kubernetes-sigs/aws-fsx-csi-driver/tree/master/examples/kubernetes/dynamic_provisioning).

``` yaml linenums="1" title="storage-class.yaml"
kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
  name: fsx-sc
provisioner: fsx.csi.aws.com
parameters:
  subnetId: subnet-0eabfaa81fb22bcaf
  securityGroupIds: sg-068000ccf82dfba88
  deploymentType: PERSISTENT_1
  automaticBackupRetentionDays: "1"
  dailyAutomaticBackupStartTime: "00:00"
  copyTagsToBackups: "true"
  perUnitStorageThroughput: "200"
  dataCompressionType: "NONE"
  weeklyMaintenanceStartTime: "7:09:00"
  fileSystemTypeVersion: "2.12"
  extraTags: "Tag1=Value1,Tag2=Value2"
mountOptions:
  - flock
```

!!! note

    You should check StorageClass's parameters in [HERE](https://github.com/kubernetes-sigs/aws-fsx-csi-driver/tree/master/examples/kubernetes/dynamic_provisioning#edit-storageclass).

``` yaml linenums="1" title="persistent-volume-claim.yaml"
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: fsx-claim
  namespace: default
spec:
  accessModes:
    - ReadWriteMany
  storageClassName: fsx-sc
  resources:
    requests:
      storage: 1200Gi
```

``` yaml linenums="1" title="pod.yaml"
apiVersion: v1
kind: Pod
metadata:
  name: fsx-app
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
      claimName: fsx-claim
```

### Dynamic Provisioning with Data Repository

!!! note

    You can see examples in [HERE](https://github.com/kubernetes-sigs/aws-fsx-csi-driver/tree/master/examples/kubernetes/dynamic_provisioning_s3).

``` yaml linenums="1" title="storage-class.yaml"
kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
  name: fsx-sc
provisioner: fsx.csi.aws.com
parameters:
  subnetId: subnet-0d7b5e117ad7b4961
  securityGroupIds: sg-05a37bfe01467059a
  s3ImportPath: s3://ml-training-data-000
  s3ExportPath: s3://ml-training-data-000/export
  deploymentType: SCRATCH_2
mountOptions:
  - flock
```

!!! note

    You should check StorageClass's parameters in [HERE](https://github.com/kubernetes-sigs/aws-fsx-csi-driver/tree/master/examples/kubernetes/dynamic_provisioning_s3#edit-storageclass).

``` yaml linenums="1" title="persistent-volume-claim.yaml"
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: fsx-claim
spec:
  accessModes:
    - ReadWriteMany
  storageClassName: fsx-sc
  resources:
    requests:
      storage: 1200Gi
```

``` yaml linenums="1" title="pod.yaml"
apiVersion: v1
kind: Pod
metadata:
  name: fsx-app
spec:
  containers:
  - name: app
    image: amazonlinux:2
    command: ["/bin/sh"]
    securityContext:
      privileged: true
    args: ["-c", "while true; do echo $(date -u) >> /data/out.txt; sleep 5; done"]
    lifecycle:
      postStart:
        exec:
          command: ["amazon-linux-extras", "install", "lustre2.10", "-y"]
    volumeMounts:
    - name: persistent-storage
      mountPath: /data
  volumes:
  - name: persistent-storage
    persistentVolumeClaim:
      claimName: fsx-claim
```