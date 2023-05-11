---
title: Enable Container Insights on EKS
description: Enable Container Insights on EKS using CloudWatch Agent and ADOT.
---

# Enable Container Insights on EKS

## Using CloudWatch Agent on EC2

### Create a service account for CloudWatch Agent

=== ":simple-linux: Linux"

    ``` bash hl_lines="1 2 3 4"
    CLUSTER_NAME="<cluster name>"
    ROLE_NAME="<role_name>"
    PROJECT_NAME="<project name>"
    REGION="<region code>"

    eksctl create iamserviceaccount \
        --cluster $CLUSTER_NAME \
        --name cloudwatch-agent \
        --namespace amazon-cloudwatch \
        --attach-policy-arn arn:aws:iam::aws:policy/CloudWatchAgentServerPolicy \
        --role-name $ROLE_NAME \
        --tags project=$PROJECT_NAME \
        --region $REGION \
        --override-existing-serviceaccounts \
        --approve
    ```

=== ":simple-windows: Windows"

    ``` powershell hl_lines="1 2 3 4"
    $CLUSTER_NAME="<cluster name>"
    $ROLE_NAME="<role_name>"
    $PROJECT_NAME="<project name>"
    $REGION="<region code>"

    eksctl create iamserviceaccount `
        --cluster $CLUSTER_NAME `
        --name cloudwatch-agent `
        --namespace amazon-cloudwatch `
        --attach-policy-arn arn:aws:iam::aws:policy/CloudWatchAgentServerPolicy `
        --role-name $ROLE_NAME `
        --tags project=$PROJECT_NAME `
        --region $REGION `
        --override-existing-serviceaccounts `
        --approve
    ```

### Deploy CloudWatch Agent using `helm`

=== ":simple-linux: Linux"

    ``` bash hl_lines="1"
    CLUSTER_NAME="<cluster name>"

    helm repo add container-insights https://marcus16-kang.github.io/aws-container-insights-chart/
    helm install cloudwatch-agent container-insights/cloudwatch-agent \
        --namespace amazon-cloudwatch \
        --set clusterName=$CLUSTER_NAME \
        --set serviceAccount.create=false
    ```

=== ":simple-windows: Windows"

    ``` powershell hl_lines="1"
    $CLUSTER_NAME="<cluster name>"

    helm repo add container-insights https://marcus16-kang.github.io/aws-container-insights-chart/
    helm install cloudwatch-agent container-insights/cloudwatch-agent `
        --namespace amazon-cloudwatch `
        --set clusterName=$CLUSTER_NAME `
        --set serviceAccount.create=false
    ```

!!! note

    If you want to use tolerations, using this parameter.

    === ":simple-linux: Linux"

        ``` bash
        --set tolerations[0].key=key1 \
        --set tolerations[0].value=value1 \
        --set tolerations[0].operator=Equal \
        --set tolerations[0].effect=NoSchedule
        ```

    === ":simple-windows: Windows"

        ``` powershell
        --set tolerations[0].key=key1 `
        --set tolerations[0].value=value1 `
        --set tolerations[0].operator=Equal `
        --set tolerations[0].effect=NoSchedule
        ```

[Github Documentation](https://github.com/marcus16-kang/aws-container-insights-chart/tree/main/cloudwatch-agent)

### Deploy CloudWatch Agent using `kubectl`

**Create a namespace for CloudWatch**

``` shell
kubectl apply -f https://raw.githubusercontent.com/aws-samples/amazon-cloudwatch-container-insights/latest/k8s-deployment-manifest-templates/deployment-mode/daemonset/container-insights-monitoring/cloudwatch-namespace.yaml
```

??? note "cloudwatch-namespace.yaml"

    ``` yaml linenums="1"
    # create amazon-cloudwatch namespace
    apiVersion: v1
    kind: Namespace
    metadata:
      name: amazon-cloudwatch
      labels:
        name: amazon-cloudwatch
    ```

[AWS Documentation](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/Container-Insights-setup-metrics.html#create-namespace-metrics)

**Create a service account for CloudWatch Agent**

``` shell
kubectl apply -f https://raw.githubusercontent.com/aws-samples/amazon-cloudwatch-container-insights/latest/k8s-deployment-manifest-templates/deployment-mode/daemonset/container-insights-monitoring/cwagent/cwagent-serviceaccount.yaml
```

??? note "cwagent-serviceaccount.yaml"

    ``` yaml linenums="1"
    # create cwagent service account and role binding
    apiVersion: v1
    kind: ServiceAccount
    metadata:
      name: cloudwatch-agent
      namespace: amazon-cloudwatch

    ---
    kind: ClusterRole
    apiVersion: rbac.authorization.k8s.io/v1
    metadata:
      name: cloudwatch-agent-role
    rules:
      - apiGroups: [""]
        resources: ["pods", "nodes", "endpoints"]
        verbs: ["list", "watch"]
      - apiGroups: ["apps"]
        resources: ["replicasets"]
        verbs: ["list", "watch"]
      - apiGroups: ["batch"]
        resources: ["jobs"]
        verbs: ["list", "watch"]
      - apiGroups: [""]
        resources: ["nodes/proxy"]
        verbs: ["get"]
      - apiGroups: [""]
        resources: ["nodes/stats", "configmaps", "events"]
        verbs: ["create"]
      - apiGroups: [""]
        resources: ["configmaps"]
        resourceNames: ["cwagent-clusterleader"]
        verbs: ["get","update"]

    ---
    kind: ClusterRoleBinding
    apiVersion: rbac.authorization.k8s.io/v1
    metadata:
      name: cloudwatch-agent-role-binding
    subjects:
      - kind: ServiceAccount
        name: cloudwatch-agent
        namespace: amazon-cloudwatch
    roleRef:
      kind: ClusterRole
      name: cloudwatch-agent-role
      apiGroup: rbac.authorization.k8s.io
    ```

[AWS Documentation](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/Container-Insights-setup-metrics.html#create-service-account)

**Create a ConfigMap for the CloudWatch agent**

=== ":simple-linux: Linux"

    ``` bash hl_lines="1"
    CLUSTER_NAME="<cluster name>"

    curl -LO https://raw.githubusercontent.com/aws-samples/amazon-cloudwatch-container-insights/latest/k8s-deployment-manifest-templates/deployment-mode/daemonset/container-insights-monitoring/cwagent/cwagent-configmap.yaml

    sed -i "s/{{cluster_name}}/$CLUSTER_NAME/" cwagent-configmap.yaml

    kubectl apply -f cwagent-configmap.yaml
    ```

=== ":simple-windows: Windows"

    ``` powershell hl_lines="1"
    $CLUSTER_NAME="<cluster name>"

    curl.exe -LO https://raw.githubusercontent.com/aws-samples/amazon-cloudwatch-container-insights/latest/k8s-deployment-manifest-templates/deployment-mode/daemonset/container-insights-monitoring/cwagent/cwagent-configmap.yaml

    (Get-Content cwagent-configmap.yaml) | ForEach-Object { $_ -replace '{{cluster_name}}', $CLUSTER_NAME } | Set-Content cwagent-configmap.yaml

    kubectl apply -f cwagent-configmap.yaml
    ```

??? note "cwagent-configmap.yaml"

    ``` yaml linenums="1"
    # create configmap for cwagent config
    apiVersion: v1
    data:
      # Configuration is in Json format. No matter what configure change you make,
      # please keep the Json blob valid.
      cwagentconfig.json: |
        {
          "logs": {
            "metrics_collected": {
              "kubernetes": {
                "cluster_name": "{{cluster_name}}",
                "metrics_collection_interval": 60
              }
            },
            "force_flush_interval": 5
          }
        }
    kind: ConfigMap
    metadata:
      name: cwagentconfig
      namespace: amazon-cloudwatch
    ```

[AWS Documentation](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/Container-Insights-setup-metrics.html#create-configmap)

**Deploy the CloudWatch agent as a DaemonSet**

``` shell
kubectl apply -f https://raw.githubusercontent.com/aws-samples/amazon-cloudwatch-container-insights/latest/k8s-deployment-manifest-templates/deployment-mode/daemonset/container-insights-monitoring/cwagent/cwagent-daemonset.yaml

kubectl get pods -n amazon-cloudwatch
```

??? note "cwagent-daemonset.yaml"

    ``` yaml linenums="1"
    # deploy cwagent as daemonset
    apiVersion: apps/v1
    kind: DaemonSet
    metadata:
      name: cloudwatch-agent
      namespace: amazon-cloudwatch
    spec:
      selector:
        matchLabels:
          name: cloudwatch-agent
      template:
        metadata:
          labels:
            name: cloudwatch-agent
        spec:
          containers:
            - name: cloudwatch-agent
              image: public.ecr.aws/cloudwatch-agent/cloudwatch-agent:1.247357.0b252275
              #ports:
              #  - containerPort: 8125
              #    hostPort: 8125
              #    protocol: UDP
              resources:
                limits:
                  cpu:  200m
                  memory: 200Mi
                requests:
                  cpu: 200m
                  memory: 200Mi
              # Please don't change below envs
              env:
                - name: HOST_IP
                  valueFrom:
                    fieldRef:
                      fieldPath: status.hostIP
                - name: HOST_NAME
                  valueFrom:
                    fieldRef:
                      fieldPath: spec.nodeName
                - name: K8S_NAMESPACE
                  valueFrom:
                    fieldRef:
                      fieldPath: metadata.namespace
                - name: CI_VERSION
                  value: "k8s/1.3.12"
              # Please don't change the mountPath
              volumeMounts:
                - name: cwagentconfig
                  mountPath: /etc/cwagentconfig
                - name: rootfs
                  mountPath: /rootfs
                  readOnly: true
                - name: dockersock
                  mountPath: /var/run/docker.sock
                  readOnly: true
                - name: varlibdocker
                  mountPath: /var/lib/docker
                  readOnly: true
                - name: containerdsock
                  mountPath: /run/containerd/containerd.sock
                  readOnly: true
                - name: sys
                  mountPath: /sys
                  readOnly: true
                - name: devdisk
                  mountPath: /dev/disk
                  readOnly: true
          nodeSelector:
            kubernetes.io/os: linux
          volumes:
            - name: cwagentconfig
              configMap:
                name: cwagentconfig
            - name: rootfs
              hostPath:
                path: /
            - name: dockersock
              hostPath:
                path: /var/run/docker.sock
            - name: varlibdocker
              hostPath:
                path: /var/lib/docker
            - name: containerdsock
              hostPath:
                path: /run/containerd/containerd.sock
            - name: sys
              hostPath:
                path: /sys
            - name: devdisk
              hostPath:
                path: /dev/disk/
          terminationGracePeriodSeconds: 60
          serviceAccountName: cloudwatch-agent
    ```

## Using ADOT on EC2

### Create a service account for ADOT on EC2

=== ":simple-linux: Linux"

    ``` bash hl_lines="1 2 3 4"
    CLUSTER_NAME="<cluster name>"
    ROLE_NAME="<role name>"
    PROJECT_NAME="<project name>"
    REGION="<region code>"

    eksctl create iamserviceaccount \
        --cluster $CLUSTER_NAME \
        --name aws-otel-sa \
        --namespace amazon-cloudwatch \
        --attach-policy-arn arn:aws:iam::aws:policy/CloudWatchAgentServerPolicy \
        --role-name $ROLE_NAME \
        --tags project=$PROJECT_NAME \
        --region $REGION \
        --override-existing-serviceaccounts \
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
        --name aws-otel-sa ` 
        --namespace amazon-cloudwatch ` 
        --attach-policy-arn arn:aws:iam::aws:policy/CloudWatchAgentServerPolicy ` 
        --role-name $ROLE_NAME ` 
        --tags project=$PROJECT_NAME ` 
        --region $REGION `
        --override-existing-serviceaccounts ` 
        --approve
    ```

### Deploy ADOT on EC2 using `helm`

=== ":simple-linux: Linux"

    ``` bash
    helm repo add container-insights https://marcus16-kang.github.io/aws-container-insights-chart/
    helm install adot-ec2 container-insights/adot-ec2 \
        --namespace amazon-cloudwatch \
        --set serviceAccount.create=false
    ```

=== ":simple-windows: Windows"

    ``` powershell
    helm repo add container-insights https://marcus16-kang.github.io/aws-container-insights-chart/
    helm install adot-ec2 container-insights/adot-ec2 `
        --namespace amazon-cloudwatch `
        --set serviceAccount.create=false
    ```

!!! note

    If you want to use tolerations, using this parameter.

    === ":simple-linux: Linux"

        ``` bash
        --set tolerations[0].key=key1 \
        --set tolerations[0].value=value1 \
        --set tolerations[0].operator=Equal \
        --set tolerations[0].effect=NoSchedule
        ```

    === ":simple-windows: Windows"

        ``` powershell
        --set tolerations[0].key=key1 `
        --set tolerations[0].value=value1 `
        --set tolerations[0].operator=Equal `
        --set tolerations[0].effect=NoSchedule
        ```

### Deploy ADOT on EC2 using `kubectl`

``` shell
kubectl apply -f https://raw.githubusercontent.com/marcus16-kang/aws-resources-example/main/scripts/eks/otel-container-insights-infra.yaml

kubectl get pods -l name=aws-otel-eks-ci -n amazon-cloudwatch
```

??? note "otel-container-insights-infra.yaml"

    ``` yaml linenums="1"
    kind: ClusterRole
    apiVersion: rbac.authorization.k8s.io/v1
    metadata:
      name: aoc-agent-role
    rules:
      - apiGroups: [""]
        resources: ["pods", "nodes", "endpoints"]
        verbs: ["list", "watch", "get"]
      - apiGroups: ["apps"]
        resources: ["replicasets"]
        verbs: ["list", "watch", "get"]
      - apiGroups: ["batch"]
        resources: ["jobs"]
        verbs: ["list", "watch"]
      - apiGroups: [""]
        resources: ["nodes/proxy"]
        verbs: ["get"]
      - apiGroups: [""]
        resources: ["nodes/stats", "configmaps", "events"]
        verbs: ["create", "get"]
      - apiGroups: [""]
        resources: ["configmaps"]
        verbs: ["update"]
      - apiGroups: [""]
        resources: ["configmaps"]
        resourceNames: ["otel-container-insight-clusterleader"]
        verbs: ["get","update", "create"]
      - apiGroups: ["coordination.k8s.io"]
        resources: ["leases"]
        verbs: ["create","get", "update"]
      - apiGroups: ["coordination.k8s.io"]
        resources: ["leases"]
        resourceNames: ["otel-container-insight-clusterleader"]
        verbs: ["get","update", "create"]

    ---
    kind: ClusterRoleBinding
    apiVersion: rbac.authorization.k8s.io/v1
    metadata:
      name: aoc-agent-role-binding
    subjects:
      - kind: ServiceAccount
        name: aws-otel-sa
        namespace: amazon-cloudwatch
    roleRef:
      kind: ClusterRole
      name: aoc-agent-role
      apiGroup: rbac.authorization.k8s.io

    ---
    apiVersion: v1
    kind: ConfigMap
    metadata:
      name: otel-agent-conf
      namespace: amazon-cloudwatch
      labels:
        app: opentelemetry
        component: otel-agent-conf
    data:
      otel-agent-config: |
        extensions:
          health_check:

        receivers:
          awscontainerinsightreceiver:

        processors:
          batch/metrics:
            timeout: 60s

        exporters:
          awsemf:
            namespace: ContainerInsights
            log_group_name: '/aws/containerinsights/{ClusterName}/performance'
            log_stream_name: '{NodeName}'
            resource_to_telemetry_conversion:
              enabled: true
            dimension_rollup_option: NoDimensionRollup
            parse_json_encoded_attr_values: [Sources, kubernetes]
            metric_declarations:
              # node metrics
              - dimensions: [[NodeName, InstanceId, ClusterName]]
                metric_name_selectors:
                  - node_cpu_utilization
                  - node_memory_utilization
                  - node_network_total_bytes
                  - node_cpu_reserved_capacity
                  - node_memory_reserved_capacity
                  - node_number_of_running_pods
                  - node_number_of_running_containers
              - dimensions: [[ClusterName]]
                metric_name_selectors:
                  - node_cpu_utilization
                  - node_memory_utilization
                  - node_network_total_bytes
                  - node_cpu_reserved_capacity
                  - node_memory_reserved_capacity
                  - node_number_of_running_pods
                  - node_number_of_running_containers
                  - node_cpu_usage_total
                  - node_cpu_limit
                  - node_memory_working_set
                  - node_memory_limit

              # pod metrics
              - dimensions: [[PodName, Namespace, ClusterName], [Service, Namespace, ClusterName], [Namespace, ClusterName], [ClusterName]]
                metric_name_selectors:
                  - pod_cpu_utilization
                  - pod_memory_utilization
                  - pod_network_rx_bytes
                  - pod_network_tx_bytes
                  - pod_cpu_utilization_over_pod_limit
                  - pod_memory_utilization_over_pod_limit
              - dimensions: [[PodName, Namespace, ClusterName], [ClusterName]]
                metric_name_selectors:
                  - pod_cpu_reserved_capacity
                  - pod_memory_reserved_capacity
              - dimensions: [[PodName, Namespace, ClusterName]]
                metric_name_selectors:
                  - pod_number_of_container_restarts

              # cluster metrics
              - dimensions: [[ClusterName]]
                metric_name_selectors:
                  - cluster_node_count
                  - cluster_failed_node_count

              # service metrics
              - dimensions: [[Service, Namespace, ClusterName], [ClusterName]]
                metric_name_selectors:
                  - service_number_of_running_pods

              # node fs metrics
              - dimensions: [[NodeName, InstanceId, ClusterName], [ClusterName]]
                metric_name_selectors:
                  - node_filesystem_utilization

              # namespace metrics
              - dimensions: [[Namespace, ClusterName], [ClusterName]]
                metric_name_selectors:
                  - namespace_number_of_running_pods

        service:
          pipelines:
            metrics:
              receivers: [awscontainerinsightreceiver]
              processors: [batch/metrics]
              exporters: [awsemf]

          extensions: [health_check]


    ---
    # create Daemonset
    apiVersion: apps/v1
    kind: DaemonSet
    metadata:
      name: aws-otel-eks-ci
      namespace: amazon-cloudwatch
    spec:
      selector:
        matchLabels:
          name: aws-otel-eks-ci
      template:
        metadata:
          labels:
            name: aws-otel-eks-ci
        spec:
          containers:
            - name: aws-otel-collector
              image: public.ecr.aws/aws-observability/aws-otel-collector:latest
              env:
                - name: K8S_NODE_NAME
                  valueFrom:
                    fieldRef:
                      fieldPath: spec.nodeName
                - name: HOST_IP
                  valueFrom:
                    fieldRef:
                      fieldPath: status.hostIP
                - name: HOST_NAME
                  valueFrom:
                    fieldRef:
                      fieldPath: spec.nodeName
                - name: K8S_NAMESPACE
                  valueFrom:
                    fieldRef:
                      fieldPath: metadata.namespace
              imagePullPolicy: Always
              command:
                - "/awscollector"
                - "--config=/conf/otel-agent-config.yaml"
              volumeMounts:
                - name: rootfs
                  mountPath: /rootfs
                  readOnly: true
                - name: dockersock
                  mountPath: /var/run/docker.sock
                  readOnly: true
                - name: containerdsock
                  mountPath: /run/containerd/containerd.sock
                - name: varlibdocker
                  mountPath: /var/lib/docker
                  readOnly: true
                - name: sys
                  mountPath: /sys
                  readOnly: true
                - name: devdisk
                  mountPath: /dev/disk
                  readOnly: true
                - name: otel-agent-config-vol
                  mountPath: /conf
              resources:
                limits:
                  cpu:  200m
                  memory: 200Mi
                requests:
                  cpu: 200m
                  memory: 200Mi
          volumes:
            - configMap:
                name: otel-agent-conf
                items:
                  - key: otel-agent-config
                    path: otel-agent-config.yaml
              name: otel-agent-config-vol
            - name: rootfs
              hostPath:
                path: /
            - name: dockersock
              hostPath:
                path: /var/run/docker.sock
            - name: varlibdocker
              hostPath:
                path: /var/lib/docker
            - name: containerdsock
              hostPath:
                path: /run/containerd/containerd.sock
            - name: sys
              hostPath:
                path: /sys
            - name: devdisk
              hostPath:
                path: /dev/disk/
          serviceAccountName: aws-otel-sa
    ```

[AWS Knowledge Center](https://aws.amazon.com/ko/premiumsupport/knowledge-center/cloudwatch-container-insights-eks-fargate/#Set_up_Container_Insights_metrics_on_your_EKS_EC2_cluster_using_ADOT)

## Using ADOT on Fargate

> If you want to use Container Insights from Fargate pod, you can only use ADOT.

### Create a service account for ADOT on Fargate

=== ":simple-linux: Linux"
    
    ``` bash hl_lines="1 2 3 4"
    CLUSTER_NAME="<cluster name>"
    ROLE_NAME="<role name>"
    PROJECT_NAME="<project name>"
    REGION="<region code>"

    eksctl create iamserviceaccount \
        --cluster=$CLUSTER_NAME \
        --name=adot-collector \
        --namespace=amazon-cloudwatch \
        --role-name=$ROLE_NAME \
        --attach-policy-arn=arn:aws:iam::aws:policy/CloudWatchAgentServerPolicy \
        --tags project=$PROJECT_NAME \
        --region=$REGION \
        --approve
    ```

=== ":simple-windows: Windows"

    ``` powershell hl_lines="1 2 3 4"
    $CLUSTER_NAME="<cluster name>"
    $ROLE_NAME="<role name>"
    $PROJECT_NAME="<project name>"
    $REGION="<region code>"

    eksctl create iamserviceaccount `
        --cluster=$CLUSTER_NAME `
        --name=adot-collector `
        --namespace=amazon-cloudwatch `
        --role-name=$ROLE_NAME `
        --attach-policy-arn=arn:aws:iam::aws:policy/CloudWatchAgentServerPolicy `
        --tags project=$PROJECT_NAME `
        --region=$REGION `
        --approve
    ```

[AWS Knowledge Center](https://aws.amazon.com/ko/premiumsupport/knowledge-center/cloudwatch-container-insights-eks-fargate/#Set_up_Container_Insights_metrics_on_an_EKS_Fargate_cluster_using_ADOT)

### Deploy ADOT on Fargate using `helm`

=== ":simple-linux: Linux"

    ``` bash hl_lines="1 2"
    CLUSTER_NAME="<cluster name>"
    REGION="<region code>"

    helm repo add container-insights https://marcus16-kang.github.io/aws-container-insights-chart/
    helm install adot-fargate container-insights/adot-fargate \
        --namespace amazon-cloudwatch \
        --set clusterName=$CLUSTER_NAME \
        --set region=$REGION \
        --set serviceAccount.create=false
    ```

=== ":simple-windows: Windows"

    ``` powershell hl_lines="1 2"
    $CLUSTER_NAME="<cluster name>"
    $REGION="<region code>"

    helm repo add container-insights https://marcus16-kang.github.io/aws-container-insights-chart/
    helm install adot-fargate container-insights/adot-fargate `
        --namespace amazon-cloudwatch `
        --set clusterName=$CLUSTER_NAME `
        --set region=$REGION `
        --set serviceAccount.create=false
    ```

### Deploy ADOT on Fargate using `kubectl`

=== ":simple-linux: Linux"
    
    ``` bash hl_lines="1 2"
    CLUSTER_NAME="<cluster name>"
    REGION="<region code>"

    curl -LO https://raw.githubusercontent.com/marcus16-kang/aws-resources-example/main/scripts/eks/otel-fargate-container-insights.yaml
    sed -i "s|YOUR-EKS-CLUSTER-NAME|$CLUSTER_NAME|g" otel-fargate-container-insights.yaml
    sed -i "s|us-east-1|$REGION|g" otel-fargate-container-insights.yaml

    kubectl apply -f otel-fargate-container-insights.yaml
    ```

=== ":simple-windows: Windows"

    ``` powershell hl_lines="1 2"
    $CLUSTER_NAME="<cluster name>"
    $REGION="<region code>"

    curl.exe -LO https://raw.githubusercontent.com/marcus16-kang/aws-resources-example/main/scripts/eks/otel-fargate-container-insights.yaml
    $content = Get-Content otel-fargate-container-insights.yaml
    $content = $content -replace "YOUR-EKS-CLUSTER-NAME", $CLUSTER_NAME
    $content = $content -replace "us-east-1", $REGION
    $content | Set-Content otel-fargate-container-insights.yaml

    kubectl apply -f otel-fargate-container-insights.yaml
    ```

??? note "otel-fargate-container-insights.yaml"

    ``` yaml linenums="1"
    ---
    kind: ClusterRole
    apiVersion: rbac.authorization.k8s.io/v1
    metadata:
      name: adotcol-admin-role
    rules:
      - apiGroups: [""]
        resources:
          - nodes
          - nodes/proxy
          - nodes/metrics
          - services
          - endpoints
          - pods
          - pods/proxy
        verbs: ["get", "list", "watch"]
      - nonResourceURLs: [ "/metrics/cadvisor"]
        verbs: ["get", "list", "watch"]

    ---
    kind: ClusterRoleBinding
    apiVersion: rbac.authorization.k8s.io/v1
    metadata:
      name: adotcol-admin-role-binding
    subjects:
      - kind: ServiceAccount
        name: adot-collector
        namespace: amazon-cloudwatch
    roleRef:
      kind: ClusterRole
      name: adotcol-admin-role
      apiGroup: rbac.authorization.k8s.io

    # collector configuration section
    # update `ClusterName=YOUR-EKS-CLUSTER-NAME` in the env variable OTEL_RESOURCE_ATTRIBUTES
    # update `region=us-east-1` in the emfexporter if you are not using us-east-1 region
    ---
    apiVersion: v1
    kind: ConfigMap
    metadata:
      name: adot-collector-config
      namespace: amazon-cloudwatch
      labels:
        app: aws-adot
        component: adot-collector-config
    data:
      adot-collector-config: |
        receivers:
          prometheus:
            config:
              global:
                scrape_interval: 1m
                scrape_timeout: 40s

              scrape_configs:
              - job_name: 'kubelets-cadvisor-metrics'
                sample_limit: 10000
                scheme: https

                kubernetes_sd_configs:
                - role: node
                tls_config:
                  ca_file: /var/run/secrets/kubernetes.io/serviceaccount/ca.crt
                bearer_token_file: /var/run/secrets/kubernetes.io/serviceaccount/token

                relabel_configs:
                  - action: labelmap
                    regex: __meta_kubernetes_node_label_(.+)
                    # Only for Kubernetes ^1.7.3.
                    # See: https://github.com/prometheus/prometheus/issues/2916
                  - target_label: __address__
                    # Changes the address to Kube API server's default address and port
                    replacement: kubernetes.default.svc:443
                  - source_labels: [__meta_kubernetes_node_name]
                    regex: (.+)
                    target_label: __metrics_path__
                    # Changes the default metrics path to kubelet's proxy cadvdisor metrics endpoint
                    replacement: /api/v1/nodes/$${1}/proxy/metrics/cadvisor
                metric_relabel_configs:
                  # extract readable container/pod name from id field
                  - action: replace
                    source_labels: [id]
                    regex: '^/machine\.slice/machine-rkt\\x2d([^\\]+)\\.+/([^/]+)\.service$'
                    target_label: rkt_container_name
                    replacement: '$${2}-$${1}'
                  - action: replace
                    source_labels: [id]
                    regex: '^/system\.slice/(.+)\.service$'
                    target_label: systemd_service_name
                    replacement: '$${1}'
        processors:
          # rename labels which apply to all metrics and are used in metricstransform/rename processor
          metricstransform/label_1:
            transforms:
              - include: .*
                match_type: regexp
                action: update
                operations:
                  - action: update_label
                    label: name
                    new_label: container_id
                  - action: update_label
                    label: kubernetes_io_hostname
                    new_label: NodeName
                  - action: update_label
                    label: eks_amazonaws_com_compute_type
                    new_label: LaunchType

          # rename container and pod metrics which we care about.
          # container metrics are renamed to `new_container_*` to differentiate them with unused container metrics
          metricstransform/rename:
            transforms:
              - include: container_spec_cpu_quota
                new_name: new_container_cpu_limit_raw
                action: insert
                match_type: regexp
                experimental_match_labels: {"container": "\\S", "LaunchType": "fargate"}
              - include: container_spec_cpu_shares
                new_name: new_container_cpu_request
                action: insert
                match_type: regexp
                experimental_match_labels: {"container": "\\S", "LaunchType": "fargate"}
              - include: container_cpu_usage_seconds_total
                new_name: new_container_cpu_usage_seconds_total
                action: insert
                match_type: regexp
                experimental_match_labels: {"container": "\\S", "LaunchType": "fargate"}
              - include: container_spec_memory_limit_bytes
                new_name: new_container_memory_limit
                action: insert
                match_type: regexp
                experimental_match_labels: {"container": "\\S", "LaunchType": "fargate"}
              - include: container_memory_cache
                new_name: new_container_memory_cache
                action: insert
                match_type: regexp
                experimental_match_labels: {"container": "\\S", "LaunchType": "fargate"}
              - include: container_memory_max_usage_bytes
                new_name: new_container_memory_max_usage
                action: insert
                match_type: regexp
                experimental_match_labels: {"container": "\\S", "LaunchType": "fargate"}
              - include: container_memory_usage_bytes
                new_name: new_container_memory_usage
                action: insert
                match_type: regexp
                experimental_match_labels: {"container": "\\S", "LaunchType": "fargate"}
              - include: container_memory_working_set_bytes
                new_name: new_container_memory_working_set
                action: insert
                match_type: regexp
                experimental_match_labels: {"container": "\\S", "LaunchType": "fargate"}
              - include: container_memory_rss
                new_name: new_container_memory_rss
                action: insert
                match_type: regexp
                experimental_match_labels: {"container": "\\S", "LaunchType": "fargate"}
              - include: container_memory_swap
                new_name: new_container_memory_swap
                action: insert
                match_type: regexp
                experimental_match_labels: {"container": "\\S", "LaunchType": "fargate"}
              - include: container_memory_failcnt
                new_name: new_container_memory_failcnt
                action: insert
                match_type: regexp
                experimental_match_labels: {"container": "\\S", "LaunchType": "fargate"}
              - include: container_memory_failures_total
                new_name: new_container_memory_hierarchical_pgfault
                action: insert
                match_type: regexp
                experimental_match_labels: {"container": "\\S", "LaunchType": "fargate", "failure_type": "pgfault", "scope": "hierarchy"}
              - include: container_memory_failures_total
                new_name: new_container_memory_hierarchical_pgmajfault
                action: insert
                match_type: regexp
                experimental_match_labels: {"container": "\\S", "LaunchType": "fargate", "failure_type": "pgmajfault", "scope": "hierarchy"}
              - include: container_memory_failures_total
                new_name: new_container_memory_pgfault
                action: insert
                match_type: regexp
                experimental_match_labels: {"container": "\\S", "LaunchType": "fargate", "failure_type": "pgfault", "scope": "container"}
              - include: container_memory_failures_total
                new_name: new_container_memory_pgmajfault
                action: insert
                match_type: regexp
                experimental_match_labels: {"container": "\\S", "LaunchType": "fargate", "failure_type": "pgmajfault", "scope": "container"}
              - include: container_fs_limit_bytes
                new_name: new_container_filesystem_capacity
                action: insert
                match_type: regexp
                experimental_match_labels: {"container": "\\S", "LaunchType": "fargate"}
              - include: container_fs_usage_bytes
                new_name: new_container_filesystem_usage
                action: insert
                match_type: regexp
                experimental_match_labels: {"container": "\\S", "LaunchType": "fargate"}
              # POD LEVEL METRICS
              - include: container_spec_cpu_quota
                new_name: pod_cpu_limit_raw
                action: insert
                match_type: regexp
                experimental_match_labels: {"image": "^$", "container": "^$", "pod": "\\S", "LaunchType": "fargate"}
              - include: container_spec_cpu_shares
                new_name: pod_cpu_request
                action: insert
                match_type: regexp
                experimental_match_labels: {"image": "^$", "container": "^$", "pod": "\\S", "LaunchType": "fargate"}
              - include: container_cpu_usage_seconds_total
                new_name: pod_cpu_usage_seconds_total
                action: insert
                match_type: regexp
                experimental_match_labels: {"image": "^$", "container": "^$", "pod": "\\S", "LaunchType": "fargate"}
              - include: container_spec_memory_limit_bytes
                new_name: pod_memory_limit
                action: insert
                match_type: regexp
                experimental_match_labels: {"image": "^$", "container": "^$", "pod": "\\S", "LaunchType": "fargate"}
              - include: container_memory_cache
                new_name: pod_memory_cache
                action: insert
                match_type: regexp
                experimental_match_labels: {"image": "^$", "container": "^$", "pod": "\\S", "LaunchType": "fargate"}
              - include: container_memory_max_usage_bytes
                new_name: pod_memory_max_usage
                action: insert
                match_type: regexp
                experimental_match_labels: {"image": "^$", "container": "^$", "pod": "\\S", "LaunchType": "fargate"}
              - include: container_memory_usage_bytes
                new_name: pod_memory_usage
                action: insert
                match_type: regexp
                experimental_match_labels: {"image": "^$", "container": "^$", "pod": "\\S", "LaunchType": "fargate"}
              - include: container_memory_working_set_bytes
                new_name: pod_memory_working_set
                action: insert
                match_type: regexp
                experimental_match_labels: {"image": "^$", "container": "^$", "pod": "\\S", "LaunchType": "fargate"}
              - include: container_memory_rss
                new_name: pod_memory_rss
                action: insert
                match_type: regexp
                experimental_match_labels: {"image": "^$", "container": "^$", "pod": "\\S", "LaunchType": "fargate"}
              - include: container_memory_swap
                new_name: pod_memory_swap
                action: insert
                match_type: regexp
                experimental_match_labels: {"image": "^$", "container": "^$", "pod": "\\S", "LaunchType": "fargate"}
              - include: container_memory_failcnt
                new_name: pod_memory_failcnt
                action: insert
                match_type: regexp
                experimental_match_labels: {"image": "^$", "container": "^$", "pod": "\\S", "LaunchType": "fargate"}
              - include: container_memory_failures_total
                new_name: pod_memory_hierarchical_pgfault
                action: insert
                match_type: regexp
                experimental_match_labels: {"image": "^$", "container": "^$", "pod": "\\S", "LaunchType": "fargate", "failure_type": "pgfault", "scope": "hierarchy"}
              - include: container_memory_failures_total
                new_name: pod_memory_hierarchical_pgmajfault
                action: insert
                match_type: regexp
                experimental_match_labels: {"image": "^$", "container": "^$", "pod": "\\S", "LaunchType": "fargate", "failure_type": "pgmajfault", "scope": "hierarchy"}
              - include: container_memory_failures_total
                new_name: pod_memory_pgfault
                action: insert
                match_type: regexp
                experimental_match_labels: {"image": "^$", "container": "^$", "pod": "\\S", "LaunchType": "fargate", "failure_type": "pgfault", "scope": "container"}
              - include: container_memory_failures_total
                new_name: pod_memory_pgmajfault
                action: insert
                match_type: regexp
                experimental_match_labels: {"image": "^$", "container": "^$", "pod": "\\S", "LaunchType": "fargate", "failure_type": "pgmajfault", "scope": "container"}
              - include: container_network_receive_bytes_total
                new_name: pod_network_rx_bytes
                action: insert
                match_type: regexp
                experimental_match_labels: {"pod": "\\S", "LaunchType": "fargate"}
              - include: container_network_receive_packets_dropped_total
                new_name: pod_network_rx_dropped
                action: insert
                match_type: regexp
                experimental_match_labels: {"pod": "\\S", "LaunchType": "fargate"}
              - include: container_network_receive_errors_total
                new_name: pod_network_rx_errors
                action: insert
                match_type: regexp
                experimental_match_labels: {"pod": "\\S", "LaunchType": "fargate"}
              - include: container_network_receive_packets_total
                new_name: pod_network_rx_packets
                action: insert
                match_type: regexp
                experimental_match_labels: {"pod": "\\S", "LaunchType": "fargate"}
              - include: container_network_transmit_bytes_total
                new_name: pod_network_tx_bytes
                action: insert
                match_type: regexp
                experimental_match_labels: {"pod": "\\S", "LaunchType": "fargate"}
              - include: container_network_transmit_packets_dropped_total
                new_name: pod_network_tx_dropped
                action: insert
                match_type: regexp
                experimental_match_labels: {"pod": "\\S", "LaunchType": "fargate"}
              - include: container_network_transmit_errors_total
                new_name: pod_network_tx_errors
                action: insert
                match_type: regexp
                experimental_match_labels: {"pod": "\\S", "LaunchType": "fargate"}
              - include: container_network_transmit_packets_total
                new_name: pod_network_tx_packets
                action: insert
                match_type: regexp
                experimental_match_labels: {"pod": "\\S", "LaunchType": "fargate"}

          # filter out only renamed metrics which we care about
          filter:
            metrics:
              include:
                match_type: regexp
                metric_names:
                  - new_container_.*
                  - pod_.*

          # convert cumulative sum datapoints to delta
          cumulativetodelta:
            include:
                metrics:
                    - new_container_cpu_usage_seconds_total
                    - pod_cpu_usage_seconds_total
                    - pod_memory_pgfault
                    - pod_memory_pgmajfault
                    - pod_memory_hierarchical_pgfault
                    - pod_memory_hierarchical_pgmajfault
                    - pod_network_rx_bytes
                    - pod_network_rx_dropped
                    - pod_network_rx_errors
                    - pod_network_rx_packets
                    - pod_network_tx_bytes
                    - pod_network_tx_dropped
                    - pod_network_tx_errors
                    - pod_network_tx_packets
                    - new_container_memory_pgfault
                    - new_container_memory_pgmajfault
                    - new_container_memory_hierarchical_pgfault
                    - new_container_memory_hierarchical_pgmajfault
                match_type: strict

          # convert delta to rate
          deltatorate:
            metrics:
              - new_container_cpu_usage_seconds_total
              - pod_cpu_usage_seconds_total
              - pod_memory_pgfault
              - pod_memory_pgmajfault
              - pod_memory_hierarchical_pgfault
              - pod_memory_hierarchical_pgmajfault
              - pod_network_rx_bytes
              - pod_network_rx_dropped
              - pod_network_rx_errors
              - pod_network_rx_packets
              - pod_network_tx_bytes
              - pod_network_tx_dropped
              - pod_network_tx_errors
              - pod_network_tx_packets
              - new_container_memory_pgfault
              - new_container_memory_pgmajfault
              - new_container_memory_hierarchical_pgfault
              - new_container_memory_hierarchical_pgmajfault

          experimental_metricsgeneration/1:
            rules:
              - name: pod_network_total_bytes
                unit: Bytes/Second
                type: calculate
                metric1: pod_network_rx_bytes
                metric2: pod_network_tx_bytes
                operation: add
              - name: pod_memory_utilization_over_pod_limit
                unit: Percent
                type: calculate
                metric1: pod_memory_working_set
                metric2: pod_memory_limit
                operation: percent
              - name: pod_cpu_usage_total
                unit: Millicore
                type: scale
                metric1: pod_cpu_usage_seconds_total
                operation: multiply
                # core to millicore: multiply by 1000
                # millicore seconds to millicore nanoseconds: multiply by 10^9
                scale_by: 1000
              - name: pod_cpu_limit
                unit: Millicore
                type: scale
                metric1: pod_cpu_limit_raw
                operation: divide
                scale_by: 100

          experimental_metricsgeneration/2:
            rules:
              - name: pod_cpu_utilization_over_pod_limit
                type: calculate
                unit: Percent
                metric1: pod_cpu_usage_total
                metric2: pod_cpu_limit
                operation: percent

          # add `Type` and rename metrics and labels
          metricstransform/label_2:
            transforms:
              - include: pod_.*
                match_type: regexp
                action: update
                operations:
                  - action: add_label
                    new_label: Type
                    new_value: "Pod"
              - include: new_container_.*
                match_type: regexp
                action: update
                operations:
                  - action: add_label
                    new_label: Type
                    new_value: Container
              - include: .*
                match_type: regexp
                action: update
                operations:
                  - action: update_label
                    label: namespace
                    new_label: Namespace
                  - action: update_label
                    label: pod
                    new_label: PodName
              - include: ^new_container_(.*)$$
                match_type: regexp
                action: update
                new_name: container_$$1

          # add cluster name from env variable and EKS metadata
          resourcedetection:
            detectors: [env, eks]

          batch:
            timeout: 60s

        # only pod level metrics in metrics format, details in https://aws-otel.github.io/docs/getting-started/container-insights/eks-fargate
        exporters:
          awsemf:
            log_group_name: '/aws/containerinsights/{ClusterName}/performance'
            log_stream_name: '{PodName}'
            namespace: 'ContainerInsights'
            region: us-east-1
            resource_to_telemetry_conversion:
              enabled: true
            eks_fargate_container_insights_enabled: true
            parse_json_encoded_attr_values: ["kubernetes"]
            dimension_rollup_option: NoDimensionRollup
            metric_declarations:
              - dimensions: [ [ClusterName, LaunchType], [ClusterName, Namespace, LaunchType], [ClusterName, Namespace, PodName, LaunchType]]
                metric_name_selectors:
                  - pod_cpu_utilization_over_pod_limit
                  - pod_cpu_usage_total
                  - pod_cpu_limit
                  - pod_memory_utilization_over_pod_limit
                  - pod_memory_working_set
                  - pod_memory_limit
                  - pod_network_rx_bytes
                  - pod_network_tx_bytes

        extensions:
          health_check:

        service:
          pipelines:
            metrics:
              receivers: [prometheus]
              processors: [metricstransform/label_1, resourcedetection, metricstransform/rename, filter, cumulativetodelta, deltatorate, experimental_metricsgeneration/1, experimental_metricsgeneration/2, metricstransform/label_2, batch]
              exporters: [awsemf]
          extensions: [health_check]

    # configure the service and the collector as a StatefulSet
    ---
    apiVersion: v1
    kind: Service
    metadata:
      name: adot-collector-service
      namespace: amazon-cloudwatch
      labels:
        app: aws-adot
        component: adot-collector
    spec:
      ports:
        - name: metrics # default endpoint for querying metrics.
          port: 8888
      selector:
        component: adot-collector
      type: ClusterIP

    ---
    apiVersion: apps/v1
    kind: StatefulSet
    metadata:
      name: adot-collector
      namespace: amazon-cloudwatch
      labels:
        app: aws-adot
        component: adot-collector
    spec:
      selector:
        matchLabels:
          app: aws-adot
          component: adot-collector
      serviceName: adot-collector-service
      template:
        metadata:
          labels:
            app: aws-adot
            component: adot-collector
        spec:
          serviceAccountName: adot-collector
          securityContext:
            fsGroup: 65534
          containers:
            - image: amazon/aws-otel-collector:latest
              name: adot-collector
              imagePullPolicy: Always
              command:
                - "/awscollector"
                - "--config=/conf/adot-collector-config.yaml"
              env:
                - name: OTEL_RESOURCE_ATTRIBUTES
                  value: "ClusterName=YOUR-EKS-CLUSTER-NAME"
              resources:
                limits:
                  cpu: 2
                  memory: 2Gi
                requests:
                  cpu: 200m
                  memory: 400Mi
              volumeMounts:
                - name: adot-collector-config-volume
                  mountPath: /conf
          volumes:
            - configMap:
                name: adot-collector-config
                items:
                  - key: adot-collector-config
                    path: adot-collector-config.yaml
              name: adot-collector-config-volume
    ---
    ```

[AWS Knowledge Center](https://aws.amazon.com/ko/premiumsupport/knowledge-center/cloudwatch-container-insights-eks-fargate/#Set_up_Container_Insights_metrics_on_an_EKS_Fargate_cluster_using_ADOT)

## Get metrics

### Pod's CPU Utilization from Fargate

``` json hl_lines="3 6"
{
    "metrics": [
        [ { "expression": "SEARCH('{ContainerInsights,PodName,ClusterName,Namespace,LaunchType} MetricName=\"pod_cpu_usage_total\" ClusterName=\"CLUSTER_NAME\" Namespace=\"NAMESPACE_NAME\" LaunchType=\"fargate\"', 'Average', 60)", "label": "", "id": "e1" } ]
    ],
    "timezone": "UTC",
    "region": "us-east-1",
    "view": "timeSeries",
    "stacked": false,
    "stat": "Average",
    "period": 60,
    "yAxis": {
        "left": {
            "label": "Percent",
            "showUnits": false
        },
        "right": {
            "showUnits": false
        }
    },
    "title": "pod_cpu_usage_total"
}
```

### Pod's memory Utilization from Fargate

``` json hl_lines="3 6"
  {
      "metrics": [
          [ { "expression": "SEARCH('{ContainerInsights,PodName,ClusterName,Namespace,LaunchType} MetricName=\"pod_cpu_usage_total\" ClusterName=\"CLUSTER_NAME\" Namespace=\"NAMESPACE_NAME\" LaunchType=\"fargate\"', 'Average', 60)", "label": "", "id": "e1" } ]
      ],
      "timezone": "UTC",
      "region": "us-east-1",
      "view": "timeSeries",
      "stacked": false,
      "stat": "Average",
      "period": 60,
      "yAxis": {
          "left": {
              "label": "Percent",
              "showUnits": false
          },
          "right": {
              "showUnits": false
          }
      },
      "title": "pod_cpu_usage_total"
  }
```