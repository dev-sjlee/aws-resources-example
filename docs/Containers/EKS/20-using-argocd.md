---
title: Using ArgoCD
description: Install ArgoCD and other addons.
---

# Using ArgoCD

## Install ArgoCD Using `helm`

=== ":simple-linux: Linux"

    ``` bash
    curl -LO https://raw.githubusercontent.com/marcus16-kang/aws-resources-example/main/scripts/eks/argocd-values.yaml
    
    helm repo add argo https://argoproj.github.io/argo-helm
    helm install argocd argo/argo-cd \
        --create-namespace \
        --namespace argocd \
        --values ./argocd-values.yaml
    ```

=== ":simple-windows: Windows"

    ``` powershell
    curl.exe -LO https://raw.githubusercontent.com/marcus16-kang/aws-resources-example/main/scripts/eks/argocd-values.yaml
    
    helm repo add argo https://argoproj.github.io/argo-helm
    helm install argocd argo/argo-cd `
        --create-namespace `
        --namespace argocd `
        --values ./argocd-values.yaml
    ```

??? note "argocd-values.yaml"

    ``` yaml linenums="1"
    configs:
      cm:
        accounts.image-updater: apiKey
        timeout.reconciliation: 60s
      rbac:
        policy.csv: |
          p, role:image-updater, applications, get, */*, allow
          p, role:image-updater, applications, update, */*, allow
          g, image-updater, role:image-updater
        policy.default: role.readonly
    redis-ha:
      enabled: false
    controller:
      replicas: 1
    server:
      autoscaling:
        enabled: false
        minReplicas: 1
    repoServer:
      autoscaling:
        enabled: false
        minReplicas: 1
    applicationSet:
      replicaCount: 1
    dex:
      enabled: false
    notifications:
      enabled: false
    ```

[Github Documentation](https://github.com/argoproj/argo-helm/tree/main/charts/argo-cd)

### Accese ArgoCD Dashboard using `port-forward`

``` shell
kubectl port-forward service/argocd-server -n argocd 8080:443
```

### Get Admin Password

=== ":simple-linux: Linux"

    ``` bash
    kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
    ```

=== ":simple-windows: Windows"

    ``` powershell
    $PASSWORD = kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}"

    [System.Text.Encoding]::UTF8.GetString([System.Convert]::FromBase64String($PASSWORD))
    ```

### Connect to ArgoCD Server using CLI

``` shell
argocd login --insecure 127.0.0.1:8080
```

> You can access dashboard using this address([https://127.0.0.1:8080](https://127.0.0.1:8080)).

??? tip

    If you want to use load balancer to access argocd using CLI, you should use **Network Load Balancer.**

    You cannot use argocd cli if you use ALB ingress.

### Sync application using ArgoCD API

``` python hl_lines="11 13 14 19"
import requests

class BearerAuth(requests.auth.AuthBase):
    def __init__(self, token):
        self.token = token
    def __call__(self, r):
        r.headers["authorization"] = "Bearer " + self.token
        return r

response = requests.post(
    'http://<argocd-url>/api/v1/session',
    json={
      'username': '<username>',
      'password': '<password>'
    }
)
token = response.json()['token']

requests.post('http://<argocd-url>/api/v1/applications/<application name>/sync', auth=BearerAuth(token))
```

## Add other Kubernetes cluster

=== ":simple-linux: Linux"

    ``` bash hl_lines="1 2"
    ARGOCD_CONTEXT_NAME="<argocd context name>"
    TARGET_CONTEXT_NAME="<target context name>"

    kubectl apply \
        -f https://raw.githubusercontent.com/marcus16-kang/aws-resources-example/main/scripts/eks/argocd-manager.yaml \
        --context $TARGET_CONTEXT_NAME

    ARN=$(kubectl config view -o jsonpath="{$.contexts[?(@.name==\"$TARGET_CONTEXT_NAME\")].context.cluster}")
    REGION=$(echo $ARN | cut -d ':' -f 4)
    CLUSTER_NAME=$(echo $ARN | cut -d '/' -f 2)

    CLUSTER_INFO=$(aws eks describe-cluster \
        --name $CLUSTER_NAME \
        --region $REGION)

    CLUSTER_URL=$(echo $CLUSTER_INFO | jq -r '.cluster.endpoint')
    CLUSTER_CA=$(echo $CLUSTER_INFO | jq -r '.cluster.certificateAuthority.data')
    TOKEN=$(kubectl get secret argocd-manager \
        -n kube-system \
        -o jsonpath="{.data.token}" \
        --context $TARGET_CONTEXT_NAME \
    | base64 -d)

    curl -LO https://raw.githubusercontent.com/marcus16-kang/aws-resources-example/main/scripts/eks/cluster-secret.yaml
    sed -i "s|CLUSTER_NAME|$CLUSTER_NAME|g" cluster-secret.yaml
    sed -i "s|TOKEN|$TOKEN|" cluster-secret.yaml
    sed -i "s|CA_DATA|$CLUSTER_CA|" cluster-secret.yaml
    sed -i "s|SERVER_URL|$CLUSTER_URL|" cluster-secret.yaml

    kubectl apply -f ./cluster-secret.yaml --context $ARGOCD_CONTEXT_NAME
    ```

=== ":simple-windows: Windows"

    ``` powershell hl_lines="1 2"
    $ARGOCD_CONTEXT_NAME="<argocd context name>"
    $TARGET_CONTEXT_NAME="<target context name>"

    kubectl apply `
        -f https://raw.githubusercontent.com/marcus16-kang/aws-resources-example/main/scripts/eks/argocd-manager.yaml `
        --context $TARGET_CONTEXT_NAME

    $ARN = kubectl config view -o jsonpath="{$.contexts[?(@.name=='$TARGET_CONTEXT_NAME')].context.cluster}"
    $REGION = $ARN.Split(':')[3]
    $CLUSTER_NAME = $ARN.Split('/')[1]

    $CLUSTER_INFO = aws eks describe-cluster `
        --name $CLUSTER_NAME `
        --region $REGION | ConvertFrom-Json

    $CLUSTER_URL = $CLUSTER_INFO.cluster.endpoint
    $CLUSTER_CA = $CLUSTER_INFO.cluster.certificateAuthority.data
    $TOKEN = kubectl get secret argocd-manager `
        -n kube-system `
        -o jsonpath="{.data.token}" `
        --context $TARGET_CONTEXT_NAME | %{[System.Text.Encoding]::UTF8.GetString([System.Convert]::FromBase64String($_))}

    curl.exe -LO https://raw.githubusercontent.com/marcus16-kang/aws-resources-example/main/scripts/eks/cluster-secret.yaml
    $yaml = Get-Content -Path ./cluster-secret.yaml
    $yaml = $yaml -replace 'CLUSTER_NAME', $CLUSTER_NAME
    $yaml = $yaml -replace 'SA_TOKEN', $TOKEN
    $yaml = $yaml -replace 'CA_DATA', $CLUSTER_CA
    $yaml = $yaml -replace 'SERVER_URL', $CLUSTER_URL
    $yaml | Out-File -Encoding utf8 ./cluster-secret.yaml

    kubectl apply -f ./cluster-secret.yaml --context $ARGOCD_CONTEXT_NAME
    ```

??? note "argocd-manager.yaml"

    ``` yaml
    apiVersion: v1
    kind: ServiceAccount
    metadata:
      name: argocd-manager
      namespace: kube-system
    ---
    apiVersion: rbac.authorization.k8s.io/v1
    kind: ClusterRole
    metadata:
      name: argocd-manager-role
    rules:
      - apiGroups:
          - '*'
        resources:
          - '*'
        verbs:
          - '*'
      - nonResourceURLs:
          - '*'
        verbs:
          - '*'
    ---
    apiVersion: rbac.authorization.k8s.io/v1
    kind: ClusterRoleBinding
    metadata:
      name: argocd-manager-role-binding
    roleRef:
      apiGroup: rbac.authorization.k8s.io
      kind: ClusterRole
      name: argocd-manager-role
    subjects:
      - kind: ServiceAccount
        name: argocd-manager
        namespace: kube-system
    ---
    apiVersion: v1
    kind: Secret
    type: kubernetes.io/service-account-token
    metadata:
      name: argocd-manager
      namespace: kube-system
      annotations:
        kubernetes.io/service-account.name: argocd-manager
    ```

??? note "cluster-secret.yaml"

    ``` yaml
    apiVersion: v1
    kind: Secret
    metadata:
      name: CLUSTER_NAME
      namespace: argocd
      labels:
        argocd.argoproj.io/secret-type: cluster
    type: Opaque
    stringData:
      config: |
        {
          "bearerToken":"TOKEN",
          "tlsClientConfig":{
            "insecure":false,
            "caData":"CA_DATA"
          }
        }
      name: CLUSTER_NAME
      server: SERVER_URL
    ```

## Install ArgoCD Image Updater using `helm`

### Create ServiceAccount using `eksctl`

=== ":simple-linux: Linux"

    ``` bash hl_lines="1 2 3 4"
    CLUSTER_NAME="<cluster name>"
    ROLE_NAME="<role name>"
    PROJECT_NAME="<project name>"
    REGION="<region code>"

    eksctl create iamserviceaccount \
        --cluster $CLUSTER_NAME \
        --name argocd-image-updater \
        --namespace argocd \
        --attach-policy-arn arn:aws:iam::aws:policy/AmazonEC2ContainerRegistryReadOnly \
        --role-name $ROLE_NAME \
        --tags project=$PROJECT_NAME \
        --region $REGION \
        --approve
    ```

=== ":simple-windows: Windows"

    ``` powershell hl_lines="1 2 3 4"
    $CLUSTER_NAME="<cluster name>"
    $ROLE_NAME="<role name>"
    $PROJECT_NAME="<project name>"
    $REGION="<region code>"

    eksctl create iamserviceaccount `
        --cluster $CLUSTER_NAME `
        --name argocd-image-updater `
        --namespace argocd `
        --attach-policy-arn arn:aws:iam::aws:policy/AmazonEC2ContainerRegistryReadOnly `
        --role-name $ROLE_NAME `
        --tags project=$PROJECT_NAME `
        --region $REGION `
        --approve
    ```

### Get ArgoCD API Key

``` shell
argocd account generate-token --account image-updater --id image-updater
```

### Install ArgoCD Image Updater

=== ":simple-linux: Linux"

    ``` bash hl_lines="1 2"
    TOKEN="<argocd token>"
    REGION="<region code>"

    ACCOUNT_ID=$(aws sts get-caller-identity --query "Account" --output text)

    curl -LO https://raw.githubusercontent.com/marcus16-kang/aws-resources-example/main/scripts/eks/argocd-image-updater-values.yaml

    sed -i "s|ARGOCD_TOKEN|$TOKEN|g" argocd-image-updater-values.yaml
    sed -i "s|ACCOUNT_ID|$ACCOUNT_ID|g" argocd-image-updater-values.yaml
    sed -i "s|REGION_CODE|$REGION|g" argocd-image-updater-values.yaml

    helm repo add argo https://argoproj.github.io/argo-helm
    helm install argocd-image-updater argo/argocd-image-updater \
        --namespace argocd \
        --values ./argocd-image-updater-values.yaml
    ```

=== ":simple-windows: Windows"

    ``` powershell hl_lines="1 2"
    $TOKEN="<argocd token>"
    $REGION="<region code>"

    $ACCOUNT_ID = aws sts get-caller-identity --query "Account" --output text
    
    curl.exe -LO https://raw.githubusercontent.com/marcus16-kang/aws-resources-example/main/scripts/eks/argocd-image-updater-values.yaml

    (Get-Content -Path argocd-image-updater-values.yaml -Raw) -replace 'ARGOCD_TOKEN', $TOKEN `
                                                            -replace 'ACCOUNT_ID', $ACCOUNT_ID `
                                                            -replace 'REGION_CODE', $REGION | Set-Content -Path argocd-image-updater-values.yaml -Encoding utf8

    helm repo add argo https://argoproj.github.io/argo-helm
    helm install argocd-image-updater argo/argocd-image-updater `
        --namespace argocd `
        --values ./argocd-image-updater-values.yaml
    ```

??? note "argocd-image-updater-values.yaml"

    ``` yaml linenums="1"
    config:
      argocd:
        grpcWeb: true
        serverAddress: "http://argocd.argocd-server"
        insecure: true
        plaintext: true
        token: ARGOCD_TOKEN
      logLevel: debug
      registries:
        - name: ECR
          api_url: "https://ACCOUNT_ID.dkr.ecr.REGION_CODE.amazonaws.com"
          prefix: "ACCOUNT_ID.dkr.ecr.REGION_CODE.amazonaws.com"
          ping: true
          insecure: false
          credentials: "ext:/scripts/auth1.sh"
          credsexpire: 10h
    authScripts:
      enabled: true
      scripts:
        auth1.sh: |
          #!/bin/sh

          aws ecr --region REGION_CODE get-authorization-token --output text --query 'authorizationData[].authorizationToken' | base64 -d
    serviceAccount:
      create: false
      name: argocd-image-updater
    ```

[Github Documentation](https://github.com/argoproj/argo-helm/tree/main/charts/argocd-image-updater)

## Install ArgoCD Rollouts using `helm`

=== ":simple-linux: Linux"

    ``` bash
    helm repo add argo https://argoproj.github.io/argo-helm
    helm install argo-rollouts argo/argo-rollouts \
        --namespace argocd \
        --set dashboard.enabled=true
    ```

=== ":simple-windows: Windows"

    ``` powershell
    helm repo add argo https://argoproj.github.io/argo-helm
    helm install argo-rollouts argo/argo-rollouts `
        --namespace argocd `
        --set dashboard.enabled=true
    ```

[Github Documentation](https://github.com/argoproj/argo-helm/tree/main/charts/argo-rollouts)

### Access Argo Rollouts Dashboard using `port-forward`

``` shell
kubectl port-forward service/argo-rollouts-dashboard -n argocd 31000:3100
```

> You can access dashboard using this address([http://127.0.0.1:31000](http://127.0.0.1:31000)).

### Install `kubectl` Plugin

=== "Linux (x86_64)"

    ``` bash
    curl -LO https://github.com/argoproj/argo-rollouts/releases/download/v1.5.1/kubectl-argo-rollouts-linux-amd64
    sudo install -o root -g root -m 0755 kubectl-argo-rollouts-linux-amd64 /usr/local/bin/kubectl-argo-rollouts
    sudo install -o root -g root -m 0755 kubectl-argo-rollouts-linux-amd64 /usr/bin/kubectl-argo-rollouts
    rm kubectl-argo-rollouts-linux-amd64
    kubectl argo rollouts version
    ```

=== "Linux (ARM64)"

    ``` bash
    curl -LO https://github.com/argoproj/argo-rollouts/releases/download/v1.5.1/kubectl-argo-rollouts-linux-arm64
    sudo install -o root -g root -m 0755 kubectl-argo-rollouts-linux-arm64 /usr/local/bin/kubectl-argo-rollouts
    sudo install -o root -g root -m 0755 kubectl-argo-rollouts-linux-arm64 /usr/bin/kubectl-argo-rollouts
    rm kubectl-argo-rollouts-linux-arm64
    kubectl argo rollouts version
    ```

=== "Windows (Executable)"

    ``` powershell
    curl.exe -LO https://github.com/argoproj/argo-rollouts/releases/download/v1.5.1/kubectl-argo-rollouts-windows-amd64
    mv kubectl-argo-rollouts-windows-amd64 kubectl-argo-rollouts.exe
    kubectl argo rollouts version
    ```