# Using Secrets Manager

## Install the ASCP

``` shell
helm repo add secrets-store-csi-driver https://kubernetes-sigs.github.io/secrets-store-csi-driver/charts
helm install -n kube-system csi-secrets-store secrets-store-csi-driver/secrets-store-csi-driver --set grpcSupportedProviders="aws" --set syncSecret.enabled="true"

helm repo add aws-secrets-manager https://aws.github.io/secrets-store-csi-driver-provider-aws
helm install -n kube-system secrets-provider-aws aws-secrets-manager/secrets-store-csi-driver-provider-aws
```

[AWS Documentation](https://docs.aws.amazon.com/secretsmanager/latest/userguide/integrating_csi_driver.html#integrating_csi_driver_install)

## Create IAM policy for accessing Secrets Manager

=== ":simple-linux: Linux"

    ``` bash hl_lines="1 2 3 5"
    POLICY_NAME="<policy name>"
    PROJECT_NAME="<project name>"
    REGION="<region>"

    SECRET_ARN="<secret arn>"

    curl -LO https://raw.githubusercontent.com/marcus16-kang/aws-resources-example/main/scripts/eks/secrets-manager-policy.json

    sed -i "s|SECRET_ARN|$SECRET_ARN|" secrets-manager-policy.json

    POLICY_ARN=$(aws iam create-policy \
            --policy-name $POLICY_NAME \
            --policy-document file://aws-load-balancer-controller-iam-policy.json \
            --query 'Policy.Arn' \
            --output text \
            # --tags Key=project,Value=$PROJECT_NAME \  # AWS CLI v2
        )
    ```

=== ":simple-windows: Windows"

    ``` powershell hl_lines="1 2 3 5"
    $POLICY_NAME="<policy name>"
    $PROJECT_NAME="<project name>"
    $REGION="<region>"

    $SECRET_ARN="<secret arn>"

    curl.exe -LO https://raw.githubusercontent.com/marcus16-kang/aws-resources-example/main/scripts/eks/secrets-manager-policy.json

    (Get-Content secrets-manager-policy.json) | Foreach-Object { $_ -replace "SECRET_ARN", "$SECRET_ARN" } | Set-Content secrets-manager-policy.json

    $POLICY_ARN = aws iam create-policy `
            --policy-name $POLICY_NAME `
            --policy-document secrets-manager-policy.json `
            --query 'Policy.Arn' `
            --output text `
            --tags Key=project,Value=$PROJECT_NAME
    ```

## Create Service Account

=== ":simple-linux: Linux"

    ``` bash hl_lines="1 2 3 4 5 6"
    CLUSTER_NAME="<cluster name>"
    SERVICE_ACCOUNT_NAME="<service account name>"
    NAMESPACE_NAME="<namespace name>"
    ROLE_NAME="<role name>"
    PROJECT_NAME="<project name>"
    REGION="<region>"

    eksctl create iamserviceaccount \
        --cluster $CLUSTER_NAME \
        --name $SERVICE_ACCOUNT_NAME \
        --namespace $NAMESPACE_NAME \
        --attach-policy-arn $POLICY_ARN \
        --role-name $ROLE_NAME \
        --tags project=$PROJECT_NAME \
        --region $REGION \
        --override-existing-serviceaccounts \
        --approve
    ```

=== ":simple-windows: Windows"

    ``` powershell hl_lines="1 2 3 4 5 6"
    $CLUSTER_NAME="<cluster name>"
    $SERVICE_ACCOUNT_NAME="<service account name>"
    $NAMESPACE_NAME="<namespace name>"
    $ROLE_NAME="<role name>"
    $PROJECT_NAME="<project name>"
    $REGION="<region>"

    eksctl create iamserviceaccount `
        --cluster $CLUSTER_NAME `
        --name $SERVICE_ACCOUNT_NAME `
        --namespace $NAMESPACE_NAME `
        --attach-policy-arn $POLICY_ARN `
        --role-name $ROLE_NAME `
        --tags project=$PROJECT_NAME `
        --region $REGION `
        --override-existing-serviceaccounts `
        --approve
    ```

[AWS Documentation](https://docs.aws.amazon.com/secretsmanager/latest/userguide/integrating_csi_driver.html#integrating_csi_driver_access)

## Use Secrets Manager's Secret Values

``` yaml title="secret-provider-class.yaml"
apiVersion: secrets-store.csi.x-k8s.io/v1
kind: SecretProviderClass
metadata:
  name: <secretproviderclass name>
  namespace: <namespace name>
spec:
  provider: aws
  secretObjects:
    - data:
        - key: DB_USERNAME
          objectName: username
        - key: DB_PASSWORD
          objectName: password
        - key: DB_ENGINE
          objectName: engine
        - key: DB_HOST
          objectName: host
        - key: DB_PORT
          objectName: port
        - key: DB_CLUSTER_IDENTIFIER
          objectName: dbClusterIdentifier
      secretName: <secret name>
      type: Opaque
  parameters:
    objects: |
        - objectName: "<secret arn>"
          objectType: "secretsmanager"
          jmesPath:
              - path: username
                objectAlias: username
              - path: password
                objectAlias: password
              - path: engine
                objectAlias: engine
              - path: host
                objectAlias: host
              - path: to_string(port)
                objectAlias: port
              - path: dbClusterIdentifier
                objectAlias: dbClusterIdentifier
```

!!! note

    You can check JMESPath in [HERE](https://jmespath.org/).

``` yaml title="pod.yaml"
apiVersion: v1
kind: Pod
metadata:
  name: <pod name>
  namespace: <namespace name>
  labels:
    app: <pod name>
spec:
  serviceAccountName: <serviceaccount name>
  containers:
    - name: busybox
      image: busybox
      command:  # override Dockerfile `ENTRYPOINT`
        - "sleep"
        - "3600"
      env:
        - name: DB_USERNAME
          valueFrom:
            secretKeyRef:
              name: <secret name>
              key: DB_USERNAME
        - name: DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: <secret name>
              key: DB_PASSWORD
        - name: DB_ENGINE
          valueFrom:
            secretKeyRef:
              name: <secret name>
              key: DB_ENGINE
        - name: DB_HOST
          valueFrom:
            secretKeyRef:
              name: <secret name>
              key: DB_HOST
        - name: DB_PORT
          valueFrom:
            secretKeyRef:
              name: <secret name>
              key: DB_PORT
        - name: DB_CLUSTER_IDENTIFIER
          valueFrom:
            secretKeyRef:
              name: <secret name>
              key: DB_CLUSTER_IDENTIFIER
      volumeMounts:
        - name: secrets-store-inline
          mountPath: "/mnt/secrets-store"
          readOnly: true
  volumes:
    - name: secrets-store-inline
      csi:
        driver: secrets-store.csi.k8s.io
        readOnly: true
        volumeAttributes:
          secretProviderClass: "<secret name>"
```

[AWS Documentation](https://docs.aws.amazon.com/secretsmanager/latest/userguide/integrating_csi_driver.html#integrating_csi_driver_mount)