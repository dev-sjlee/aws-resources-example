---
title: Using App Mesh
description: Using App Mesh in EKS Kubernetes.
---

# Using App Mesh

## Install App Mesh

### Check Installation Instructions

``` bash
curl -L -o pre_upgrade_check.sh https://raw.githubusercontent.com/aws/eks-charts/master/stable/appmesh-controller/upgrade/pre_upgrade_check.sh
sh ./pre_upgrade_check.sh
```

### Install the CRD

``` bash
kubectl apply -k "https://github.com/aws/eks-charts/stable/appmesh-controller/crds?ref=master"
```

### Create the Namespace

``` bash
kubectl create ns appmesh-system
```

### Create the Service Account

``` bash hl_lines="1 2 3 4"
CLUSTER_NAME="<cluster name>"
ROLE_NAME="<role name>"
PROJECT_NAME="<project name>"
REGION="<region code>"

eksctl create iamserviceaccount \
    --cluster $CLUSTER_NAME \
    --namespace appmesh-system \
    --name appmesh-controller \
    --attach-policy-arn arn:aws:iam::aws:policy/AWSCloudMapFullAccess,arn:aws:iam::aws:policy/AWSAppMeshFullAccess \
    --role-name $ROLE_NAME \
    --tags project=$PROJECT_NAME \
    --region $REGION \
    --override-existing-serviceaccounts \
    --approve
```

## Deploy using `helm`

=== "x86_64"

    ``` bash
    REGION="<region code>"

    helm repo add eks https://aws.github.io/eks-charts
    helm upgrade -i appmesh-controller eks/appmesh-controller \
        --namespace appmesh-system \
        --set region=$REGION \
        --set serviceAccount.create=false \
        --set serviceAccount.name=appmesh-controller
    ```

=== "ARM64"

    ``` bash
    REGION="<region code>"

    helm repo add eks https://aws.github.io/eks-charts
    helm upgrade -i appmesh-controller eks/appmesh-controller \
        --namespace appmesh-system \
        --set region=$REGION \
        --set serviceAccount.create=false \
        --set serviceAccount.name=appmesh-controller \
        --set image.tag=v1.11.0-linux_arm64
    ```

!!! note

    Do you want to tracing, use these options.

    ``` bash
    --set tracing.enabled=true \
    --set tracing.provider=x-ray
    ```

[AWS Documentation](https://docs.aws.amazon.com/app-mesh/latest/userguide/getting-started-kubernetes.html#install-controller)

## Deploy App Mesh Resources

### App Mesh Namespace

**Namespace manifest resource**

``` yaml title="namespace.yaml" linenums="1" hl_lines="4 6"
apiVersion: v1
kind: Namespace
metadata:
  name: my-apps
  labels:
    mesh: my-mesh
    appmesh.k8s.aws/sidecarInjectorWebhook: enabled
```

**Deploy namespace resource**

``` bash
kubectl apply -f namespace.yaml
```

### App Mesh service mesh

**Mesh manifest**

``` yaml title="mesh.yaml" linenums="1" hl_lines="4 8"
apiVersion: appmesh.k8s.aws/v1beta2
kind: Mesh
metadata:
  name: my-mesh
spec:
  namespaceSelector:
    matchLabels:
      mesh: my-mesh
```

**Deploy mesh resource**

``` bash
kubectl apply -f mesh.yaml
```

**Show kubernetes mesh resource**

``` bash hl_lines="1"
kubectl describe mesh my-mesh
```

**Show service mesh**

``` bash hl_lines="1"
aws appmesh describe-mesh --mesh-name my-mesh --region region-code
```

### App Mesh virtual node

**Virtual node manifest**

``` yaml title="virtual-node.yaml" linenums="1" hl_lines="4 5 9 12 13 16"
apiVersion: appmesh.k8s.aws/v1beta2
kind: VirtualNode
metadata:
  name: my-service-a
  namespace: my-apps
spec:
  podSelector:
    matchLabels:
      app: my-app-1
  listeners:
    - portMapping:
        port: 80
        protocol: http
  serviceDiscovery:
    dns:
      hostname: my-service-a.my-apps.svc.cluster.local
```

**See spec**

``` bash
aws appmesh create-virtual-node --generate-cli-skeleton yaml-input
```

**Deploy virtual node resource**

``` bash
kubectl apply -f virtual-node.yaml
```

**Show kubernetes virtual node resource**

``` bash hl_lines="1"
kubectl describe virtualnode my-service-a -n my-apps
```

**Show virtual node**

``` bash hl_lines="1"
aws appmesh describe-virtual-node --mesh-name my-mesh --virtual-node-name my-service-a_my-apps
```

### App Mesh virtual router

**Virtual router manifest**

``` yaml title="virtual-router.yaml" linenums="1" hl_lines="4 5 9 10 12 15 19 20"
apiVersion: appmesh.k8s.aws/v1beta2
kind: VirtualRouter
metadata:
  namespace: my-apps
  name: my-service-a-virtual-router
spec:
  listeners:
    - portMapping:
        port: 80
        protocol: http
  routes:
    - name: my-service-a-route
      httpRoute:
        match:
          prefix: /
        action:
          weightedTargets:
            - virtualNodeRef:
                name: my-service-a
              weight: 1
```

**See virtual router spec**

``` bash
aws appmesh create-virtual-router --generate-cli-skeleton yaml-input
```

**See router spec**

``` bash
aws appmesh create-route --generate-cli-skeleton yaml-input
```

**Deploy virtual router resource**

``` bash
kubectl apply -f virtual-router.yaml
```

**Show kubernetes virtual router resource**

``` bash hl_lines="1"
kubectl describe virtualrouter my-service-a-virtual-router -n my-apps
```

**Show virtual router**

``` bash hl_lines="1"
aws appmesh describe-virtual-router --virtual-router-name my-service-a-virtual-router_my-apps --mesh-name my-mesh
```

**Show router**

``` bash hl_lines="1 2 3 4"
aws appmesh describe-route \
    --route-name my-service-a-route \
    --virtual-router-name my-service-a-virtual-router_my-apps \
    --mesh-name my-mesh
```