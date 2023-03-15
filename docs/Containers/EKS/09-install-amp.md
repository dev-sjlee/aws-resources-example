# Install Amazon Managed Prometheus

!!! note

    Maybe you need EBS CSI Driver to use AMP. [Using EBS](../15-using-ebs/)

## Create prometheus workspace

=== ":simple-linux: Linux"

    ``` bash hl_lines="1 2"
    WORKSPACE_NAME="<amp workspace name>"
    REGION="<region code>"

    aws amp create-workspace \
        --alias $WORKSPACE_NAME \
        --region $REGION
    ```

=== ":simple-windows: Windows"

    ``` powershell hl_lines="1 2"
    $WORKSPACE_NAME="<amp workspace name>"
    $REGION="<region code>"

    aws amp create-workspace `
        --alias $WORKSPACE_NAME `
        --region $REGION
    ```

[AWS Documentation](https://awscli.amazonaws.com/v2/documentation/api/latest/reference/amp/create-workspace.html)

## Create prometheus IAM role for service account

=== ":simple-linux: Linux"

    ``` bash hl_lines="1 2 3"
    CLUSTER_NAME="<cluster name>"
    WORKSPACE_NAME="<amp workspace name>"
    REGION="<region code>"

    curl -O https://raw.githubusercontent.com/marcus16-kang/aws-resources-example/main/scripts/eks/createIRSA-AMPIngest.sh
    chmod +x createIRSA-AMPIngest.sh

    ./createIRSA-AMPIngest.sh $CLUSTER_NAME $WORKSPACE_NAME $REGION
    ```

=== ":simple-windows: Windows"

[AWS Documentation](https://docs.aws.amazon.com/prometheus/latest/userguide/set-up-irsa.html#set-up-irsa-ingest)

## Install prometheus with `helm`

``` shell hl_lines="5 12 15 17 24 25"
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo add kube-state-metrics https://kubernetes.github.io/kube-state-metrics
helm repo update

kubectl create namespace <prometheus namespace>

cat << EOF >> amp-helm-conf.yaml
serviceAccounts:
    server:
        name: "amp-iamproxy-ingest-service-account"
        annotations:
            eks.amazonaws.com/role-arn: "<prometheus iam role arn>"
server:
    remoteWrite:
        - url: <amp remote write url>
          sigv4:
            region: <region code>
          queue_config:
            max_samples_per_send: 1000
            max_shards: 200
            capacity: 2500
EOF

helm install <prometheus chart name> prometheus-community/prometheus \
    -n <prometheus namespace> \
    -f amp-helm-conf.yaml
```
[AWS Documentation](https://docs.aws.amazon.com/prometheus/latest/userguide/AMP-onboard-ingest-metrics-new-Prometheus.html)