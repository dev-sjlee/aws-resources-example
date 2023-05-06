# Enable Logging with Fluent Bit

## EC2

### Create a service account for Fluent Bit

=== ":simple-linux: Linux"

    ``` bash hl_lines="1 2 3 4"
    CLUSTER_NAME="<cluster name>"
    ROLE_NAME="<role_name>"
    PROJECT_NAME="<project name>"
    REGION="<region code>"

    eksctl create iamserviceaccount \
        --cluster $CLUSTER_NAME \
        --name fluent-bit \
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
        --name fluent-bit `
        --namespace amazon-cloudwatch `
        --attach-policy-arn arn:aws:iam::aws:policy/CloudWatchAgentServerPolicy `
        --role-name $ROLE_NAME `
        --tags project=$PROJECT_NAME `
        --region $REGION `
        --override-existing-serviceaccounts `
        --approve
    ```

### Deploy Fluent Bit using `helm`

=== ":simple-linux: Linux"

    ``` bash hl_lines="1 2"
    CLUSTER_NAME="<cluster name>"
    REGION="<region code>"

    FluentBitHttpPort='2020'
    FluentBitReadFromHead='Off'
    [[ ${FluentBitReadFromHead} = 'On' ]] && FluentBitReadFromTail='Off'|| FluentBitReadFromTail='On'
    [[ -z ${FluentBitHttpPort} ]] && FluentBitHttpServer='Off' || FluentBitHttpServer='On'

    helm repo add container-insights https://marcus16-kang.github.io/aws-container-insights-chart/
    helm install fluent-bit container-insights/fluent-bit \
        --namespace amazon-cloudwatch \
        --set clusterInfo.clusterName=$CLUSTER_NAME \
        --set clusterInfo.region=$REGION \
        --set clusterInfo.fluentBitHttpServer=$FluentBitHttpServer \
        --set clusterInfo.fluentBitHttpPort=$FluentBitHttpPort \
        --set clusterInfo.fluentBitReadHead=$REGFluentBitReadFromHeadION \
        --set clusterInfo.fluentBitReadTail=$FluentBitReadFromTail \
        --set serviceAccount.create=false
    ```

=== ":simple-windows: Windows"

    ``` powershell hl_lines="1 2"
    $CLUSTER_NAME="<cluster name>"
    $REGION="<region code>"

    $FluentBitHttpPort='2020'
    $FluentBitReadFromHead='Off'
    
    if ($FluentBitReadFromHead -eq 'On') {
        $FluentBitReadFromTail = 'Off'
    } else {
        $FluentBitReadFromTail = 'On'
    }
    if (-not $FluentBitHttpPort) {
        $FluentBitHttpServer = 'Off'
    } else {
        $FluentBitHttpServer = 'On'
    }

    helm repo add container-insights https://marcus16-kang.github.io/aws-container-insights-chart/
    helm install fluent-bit container-insights/fluent-bit `
        --namespace amazon-cloudwatch `
        --set clusterInfo.clusterName=$CLUSTER_NAME `
        --set clusterInfo.region=$REGION `
        --set clusterInfo.fluentBitHttpServer=$FluentBitHttpServer `
        --set clusterInfo.fluentBitHttpPort=$FluentBitHttpPort `
        --set clusterInfo.fluentBitReadHead=$REGFluentBitReadFromHeadION `
        --set clusterInfo.fluentBitReadTail=$FluentBitReadFromTail `
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

!!! tip

    Do you want to log with custom cloudwatch log group, please add some config like this in helm chart value.

    ``` yaml title="fluent-bit-values.yaml" linenums="1" hl_lines="3 4"
    clusterInfo:
      configMapName: "fluent-bit-cluster-info"
      clusterName: "" # [REQUIRED]
      region: ""      # [REQUIRED]
      fluentBitHttpServer: "On"
      fluentBitHttpPort: "2020"
      fluentBitReadHead: "Off"
      fluentBitReadTail: "On"
    
    fluentBitConfig:
      configMapName: "fluent-bit-config"
      extraApplicationConfig: |
        [INPUT]
            Name                tail
            Tag                 application.app_name
            Path                /var/log/containers/app_name-*
            multiline.parser    docker, cri
            DB                  /var/fluent-bit/state/flb_app_name.db
            Mem_Buf_Limit       50MB
            Skip_Long_Lines     On
            Refresh_Interval    10
            Rotate_Wait         30
            storage.type        filesystem
            Read_from_Head      ${READ_FROM_HEAD}
        
        [OUTPUT]
            Name                cloudwatch_logs
            Match               application.app_name
            region              ${AWS_REGION}
            log_group_name      /aws/${AWS_REGION}/app_name
            log_stream_prefix   ${HOST_NAME}-
            auto_create_group   true
            extra_user_agent    container-insights
    
    serviceAccount:
      create: false
    
    tolerations: []
    # - key: "key1"         # taint key
    #     value: "value1"     # taint value
    #     operator: "Equal"
    #     effect: "NoSchedule"
    ```

[Github Documentation](https://github.com/marcus16-kang/aws-container-insights-chart/tree/main/fluent-bit)

### Deploy Fluent Bit using `kubectl`

=== ":simple-linux: Linux"

    ``` bash hl_lines="1 2"
    CLUSTER_NAME="<cluster name>"
    REGION="<region code>"
    FluentBitHttpPort='2020'
    FluentBitReadFromHead='Off'
    [[ ${FluentBitReadFromHead} = 'On' ]] && FluentBitReadFromTail='Off'|| FluentBitReadFromTail='On'
    [[ -z ${FluentBitHttpPort} ]] && FluentBitHttpServer='Off' || FluentBitHttpServer='On'
    kubectl create configmap fluent-bit-cluster-info \
        --from-literal=cluster.name=${CLUSTER_NAME} \
        --from-literal=http.server=${FluentBitHttpServer} \
        --from-literal=http.port=${FluentBitHttpPort} \
        --from-literal=read.head=${FluentBitReadFromHead} \
        --from-literal=read.tail=${FluentBitReadFromTail} \
        --from-literal=logs.region=${REGION} -n amazon-cloudwatch

    kubectl apply -f https://raw.githubusercontent.com/marcus16-kang/aws-resources-example/main/scripts/eks/fluent-bit.yaml

    kubectl get pods -n amazon-cloudwatch
    ```

=== ":simple-windows: Windows"

    ``` powershell hl_lines="1 2"
    $CLUSTER_NAME="<cluster name>"
    $REGION="<region code>"
    $FluentBitHttpPort="2020"
    $FluentBitReadFromHead="Off"
    
    if ($FluentBitReadFromHead -eq 'On') {
        $FluentBitReadFromTail = 'Off'
    } else {
        $FluentBitReadFromTail = 'On'
    }
    if (-not $FluentBitHttpPort) {
        $FluentBitHttpServer = 'Off'
    } else {
        $FluentBitHttpServer = 'On'
    }

    kubectl create configmap fluent-bit-cluster-info `
        --from-literal=cluster.name=${CLUSTER_NAME} `
        --from-literal=http.server=${FluentBitHttpServer} `
        --from-literal=http.port=${FluentBitHttpPort} `
        --from-literal=read.head=${FluentBitReadFromHead} `
        --from-literal=read.tail=${FluentBitReadFromTail} `
        --from-literal=logs.region=${REGION} -n amazon-cloudwatch

    kubectl apply -f https://raw.githubusercontent.com/marcus16-kang/aws-resources-example/main/scripts/eks/fluent-bit.yaml

    kubectl get pods -n amazon-cloudwatch
    ```

??? note "fluent-bit.yaml"

    ``` yaml linenums="1"
    # apiVersion: v1
    # kind: ServiceAccount
    # metadata:
    #   name: fluent-bit
    #   namespace: amazon-cloudwatch
    ---
    apiVersion: rbac.authorization.k8s.io/v1
    kind: ClusterRole
    metadata:
      name: fluent-bit-role
    rules:
      - nonResourceURLs:
          - /metrics
        verbs:
          - get
      - apiGroups: [""]
        resources:
          - namespaces
          - pods
          - pods/logs
          - nodes
          - nodes/proxy
        verbs: ["get", "list", "watch"]
    ---
    apiVersion: rbac.authorization.k8s.io/v1
    kind: ClusterRoleBinding
    metadata:
      name: fluent-bit-role-binding
    roleRef:
      apiGroup: rbac.authorization.k8s.io
      kind: ClusterRole
      name: fluent-bit-role
    subjects:
      - kind: ServiceAccount
        name: fluent-bit
        namespace: amazon-cloudwatch
    ---
    apiVersion: v1
    kind: ConfigMap
    metadata:
      name: fluent-bit-config
      namespace: amazon-cloudwatch
      labels:
        k8s-app: fluent-bit
    data:
      fluent-bit.conf: |
        [SERVICE]
            Flush                     5
            Grace                     30
            Log_Level                 info
            Daemon                    off
            Parsers_File              parsers.conf
            HTTP_Server               ${HTTP_SERVER}
            HTTP_Listen               0.0.0.0
            HTTP_Port                 ${HTTP_PORT}
            storage.path              /var/fluent-bit/state/flb-storage/
            storage.sync              normal
            storage.checksum          off
            storage.backlog.mem_limit 5M
            
        @INCLUDE application-log.conf
        @INCLUDE dataplane-log.conf
        @INCLUDE host-log.conf
      
      application-log.conf: |
        [INPUT]
            Name                tail
            Tag                 application.*
            Exclude_Path        /var/log/containers/cloudwatch-agent*, /var/log/containers/fluent-bit*, /var/log/containers/aws-node*, /var/log/containers/kube-proxy*
            Path                /var/log/containers/*.log
            multiline.parser    docker, cri
            DB                  /var/fluent-bit/state/flb_container.db
            Mem_Buf_Limit       50MB
            Skip_Long_Lines     On
            Refresh_Interval    10
            Rotate_Wait         30
            storage.type        filesystem
            Read_from_Head      ${READ_FROM_HEAD}

        [INPUT]
            Name                tail
            Tag                 application.*
            Path                /var/log/containers/fluent-bit*
            multiline.parser    docker, cri
            DB                  /var/fluent-bit/state/flb_log.db
            Mem_Buf_Limit       5MB
            Skip_Long_Lines     On
            Refresh_Interval    10
            Read_from_Head      ${READ_FROM_HEAD}

        [INPUT]
            Name                tail
            Tag                 application.*
            Path                /var/log/containers/cloudwatch-agent*
            multiline.parser    docker, cri
            DB                  /var/fluent-bit/state/flb_cwagent.db
            Mem_Buf_Limit       5MB
            Skip_Long_Lines     On
            Refresh_Interval    10
            Read_from_Head      ${READ_FROM_HEAD}

        [FILTER]
            Name                kubernetes
            Match               application.*
            Kube_URL            https://kubernetes.default.svc:443
            Kube_Tag_Prefix     application.var.log.containers.
            Merge_Log           On
            Merge_Log_Key       log_processed
            K8S-Logging.Parser  On
            K8S-Logging.Exclude Off
            Labels              Off
            Annotations         Off
            Use_Kubelet         On
            Kubelet_Port        10250
            Buffer_Size         0

        [OUTPUT]
            Name                cloudwatch_logs
            Match               application.*
            region              ${AWS_REGION}
            log_group_name      /aws/containerinsights/${CLUSTER_NAME}/application
            log_stream_prefix   ${HOST_NAME}-
            auto_create_group   true
            extra_user_agent    container-insights

      dataplane-log.conf: |
        [INPUT]
            Name                systemd
            Tag                 dataplane.systemd.*
            Systemd_Filter      _SYSTEMD_UNIT=docker.service
            Systemd_Filter      _SYSTEMD_UNIT=kubelet.service
            DB                  /var/fluent-bit/state/systemd.db
            Path                /var/log/journal
            Read_From_Tail      ${READ_FROM_TAIL}

        [INPUT]
            Name                tail
            Tag                 dataplane.tail.*
            Path                /var/log/containers/aws-node*, /var/log/containers/kube-proxy*
            multiline.parser    docker, cri
            DB                  /var/fluent-bit/state/flb_dataplane_tail.db
            Mem_Buf_Limit       50MB
            Skip_Long_Lines     On
            Refresh_Interval    10
            Rotate_Wait         30
            storage.type        filesystem
            Read_from_Head      ${READ_FROM_HEAD}

        [FILTER]
            Name                modify
            Match               dataplane.systemd.*
            Rename              _HOSTNAME                   hostname
            Rename              _SYSTEMD_UNIT               systemd_unit
            Rename              MESSAGE                     message
            Remove_regex        ^((?!hostname|systemd_unit|message).)*$

        [FILTER]
            Name                aws
            Match               dataplane.*
            imds_version        v2

        [OUTPUT]
            Name                cloudwatch_logs
            Match               dataplane.*
            region              ${AWS_REGION}
            log_group_name      /aws/containerinsights/${CLUSTER_NAME}/dataplane
            log_stream_prefix   ${HOST_NAME}-
            auto_create_group   true
            extra_user_agent    container-insights

      host-log.conf: |
        [INPUT]
            Name                tail
            Tag                 host.dmesg
            Path                /var/log/dmesg
            Key                 message
            DB                  /var/fluent-bit/state/flb_dmesg.db
            Mem_Buf_Limit       5MB
            Skip_Long_Lines     On
            Refresh_Interval    10
            Read_from_Head      ${READ_FROM_HEAD}

        [INPUT]
            Name                tail
            Tag                 host.messages
            Path                /var/log/messages
            Parser              syslog
            DB                  /var/fluent-bit/state/flb_messages.db
            Mem_Buf_Limit       5MB
            Skip_Long_Lines     On
            Refresh_Interval    10
            Read_from_Head      ${READ_FROM_HEAD}

        [INPUT]
            Name                tail
            Tag                 host.secure
            Path                /var/log/secure
            Parser              syslog
            DB                  /var/fluent-bit/state/flb_secure.db
            Mem_Buf_Limit       5MB
            Skip_Long_Lines     On
            Refresh_Interval    10
            Read_from_Head      ${READ_FROM_HEAD}

        [FILTER]
            Name                aws
            Match               host.*
            imds_version        v2

        [OUTPUT]
            Name                cloudwatch_logs
            Match               host.*
            region              ${AWS_REGION}
            log_group_name      /aws/containerinsights/${CLUSTER_NAME}/host
            log_stream_prefix   ${HOST_NAME}.
            auto_create_group   true
            extra_user_agent    container-insights

      parsers.conf: |
        [PARSER]
            Name                syslog
            Format              regex
            Regex               ^(?<time>[^ ]* {1,2}[^ ]* [^ ]*) (?<host>[^ ]*) (?<ident>[a-zA-Z0-9_\/\.\-]*)(?:\[(?<pid>[0-9]+)\])?(?:[^\:]*\:)? *(?<message>.*)$
            Time_Key            time
            Time_Format         %b %d %H:%M:%S

        [PARSER]
            Name                container_firstline
            Format              regex
            Regex               (?<log>(?<="log":")\S(?!\.).*?)(?<!\\)".*(?<stream>(?<="stream":").*?)".*(?<time>\d{4}-\d{1,2}-\d{1,2}T\d{2}:\d{2}:\d{2}\.\w*).*(?=})
            Time_Key            time
            Time_Format         %Y-%m-%dT%H:%M:%S.%LZ

        [PARSER]
            Name                cwagent_firstline
            Format              regex
            Regex               (?<log>(?<="log":")\d{4}[\/-]\d{1,2}[\/-]\d{1,2}[ T]\d{2}:\d{2}:\d{2}(?!\.).*?)(?<!\\)".*(?<stream>(?<="stream":").*?)".*(?<time>\d{4}-\d{1,2}-\d{1,2}T\d{2}:\d{2}:\d{2}\.\w*).*(?=})
            Time_Key            time
            Time_Format         %Y-%m-%dT%H:%M:%S.%LZ
    ---
    apiVersion: apps/v1
    kind: DaemonSet
    metadata:
      name: fluent-bit
      namespace: amazon-cloudwatch
      labels:
        k8s-app: fluent-bit
        version: v1
        kubernetes.io/cluster-service: "true"
    spec:
      selector:
        matchLabels:
          k8s-app: fluent-bit
      template:
        metadata:
          labels:
            k8s-app: fluent-bit
            version: v1
            kubernetes.io/cluster-service: "true"
        spec:
          containers:
          - name: fluent-bit
            image: public.ecr.aws/aws-observability/aws-for-fluent-bit:stable
            imagePullPolicy: Always
            env:
                - name: AWS_REGION
                  valueFrom:
                    configMapKeyRef:
                      name: fluent-bit-cluster-info
                      key: logs.region
                - name: CLUSTER_NAME
                  valueFrom:
                    configMapKeyRef:
                      name: fluent-bit-cluster-info
                      key: cluster.name
                - name: HTTP_SERVER
                  valueFrom:
                    configMapKeyRef:
                      name: fluent-bit-cluster-info
                      key: http.server
                - name: HTTP_PORT
                  valueFrom:
                    configMapKeyRef:
                      name: fluent-bit-cluster-info
                      key: http.port
                - name: READ_FROM_HEAD
                  valueFrom:
                    configMapKeyRef:
                      name: fluent-bit-cluster-info
                      key: read.head
                - name: READ_FROM_TAIL
                  valueFrom:
                    configMapKeyRef:
                      name: fluent-bit-cluster-info
                      key: read.tail
                - name: HOST_NAME
                  valueFrom:
                    fieldRef:
                      fieldPath: spec.nodeName
                - name: HOSTNAME
                  valueFrom:
                    fieldRef:
                      apiVersion: v1
                      fieldPath: metadata.name
                - name: CI_VERSION
                  value: "k8s/1.3.13"
            resources:
                limits:
                  memory: 200Mi
                requests:
                  cpu: 500m
                  memory: 100Mi
            volumeMounts:
            # Please don't change below read-only permissions
            - name: fluentbitstate
              mountPath: /var/fluent-bit/state
            - name: varlog
              mountPath: /var/log
              readOnly: true
            - name: varlibdockercontainers
              mountPath: /var/lib/docker/containers
              readOnly: true
            - name: fluent-bit-config
              mountPath: /fluent-bit/etc/
            - name: runlogjournal
              mountPath: /run/log/journal
              readOnly: true
            - name: dmesg
              mountPath: /var/log/dmesg
              readOnly: true
          terminationGracePeriodSeconds: 10
          hostNetwork: true
          dnsPolicy: ClusterFirstWithHostNet
          volumes:
          - name: fluentbitstate
            hostPath:
              path: /var/fluent-bit/state
          - name: varlog
            hostPath:
              path: /var/log
          - name: varlibdockercontainers
            hostPath:
              path: /var/lib/docker/containers
          - name: fluent-bit-config
            configMap:
              name: fluent-bit-config
          - name: runlogjournal
            hostPath:
              path: /run/log/journal
          - name: dmesg
            hostPath:
              path: /var/log/dmesg
          serviceAccountName: fluent-bit
          tolerations:
          - key: node-role.kubernetes.io/master
            operator: Exists
            effect: NoSchedule
          - operator: "Exists"
            effect: "NoExecute"
          - operator: "Exists"
            effect: "NoSchedule"
    ```

!!! tip

    Do you want to log with custom cloudwatch log group, please add some config like this in `fluent-bit-config` configmap.

    ``` conf
    [INPUT]
        Name                tail
        Tag                 application.app_name
        Path                /var/log/containers/app_name-*
        multiline.parser    docker, cri
        DB                  /var/fluent-bit/state/flb_app_name.db
        Mem_Buf_Limit       50MB
        Skip_Long_Lines     On
        Refresh_Interval    10
        Rotate_Wait         30
        storage.type        filesystem
        Read_from_Head      ${READ_FROM_HEAD}
    
    [OUTPUT]
        Name                cloudwatch_logs
        Match               application.app_name
        region              ${AWS_REGION}
        log_group_name      /aws/${AWS_REGION}/app_name
        log_stream_prefix   ${HOST_NAME}-
        auto_create_group   true
        extra_user_agent    container-insights
    ```

[AWS Documentation](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/Container-Insights-setup-logs-FluentBit.html#Container-Insights-FluentBit-setup)

## Fargate

### Create Namespace

``` shell
kubectl apply -f https://raw.githubusercontent.com/marcus16-kang/aws-resources-example/main/scripts/eks/aws-observability-namespace.yaml
```

??? note "aws-observability-namespace.yaml"

    ``` yaml title="aws-observability-namespace.yaml" linenums="1"
    apiVersion: v1
    kind: Namespace
    metadata:
        name: aws-observability
        labels:
            aws-observability: enabled
    ```

### Create ConfigMap

=== "CloudWatch Logs"

    ``` yaml title="aws-logging-cloudwatch-logs-config.yaml" linenums="1" hl_lines="25 26 27"
    kind: ConfigMap
    apiVersion: v1
    metadata:
      name: aws-logging
      namespace: aws-observability
    data:
      flb_log_cw: "false"  # Set to true to ship Fluent Bit process logs to CloudWatch.
      filters.conf: |
        [FILTER]
            Name parser
            Match *
            Key_name log
            Parser crio
        [FILTER]
            Name kubernetes
            Match kube.*
            Merge_Log On
            Keep_Log Off
            Buffer_Size 0
            Kube_Meta_Cache_TTL 300s
      output.conf: |
        [OUTPUT]
            Name cloudwatch_logs
            Match   kube.*
            region region-code
            log_group_name my-logs
            log_stream_prefix from-fluent-bit-
            log_retention_days 60
            auto_create_group true
      parsers.conf: |
        [PARSER]
            Name crio
            Format Regex
            Regex ^(?<time>[^ ]+) (?<stream>stdout|stderr) (?<logtag>P|F) (?<log>.*)$
            Time_Key    time
            Time_Format %Y-%m-%dT%H:%M:%S.%L%z
    ```

    [Fluent Bit Documentation](https://docs.fluentbit.io/manual/pipeline/outputs/cloudwatch)

=== "ElasticSearch/OpenSearch"

    ``` yaml title="aws-logging-opensearch-config.yaml" linenums="1" hl_lines="11 13 14 16"
    kind: ConfigMap
    apiVersion: v1
    metadata:
      name: aws-logging
      namespace: aws-observability
    data:
      output.conf: |
        [OUTPUT]
            Name  es
            Match *
            Host  search-example-gjxdcilagiprbglqn42jsty66y.region-code.es.amazonaws.com
            Port  443
            Index example
            Type  example_type
            AWS_Auth On
            AWS_Region region-code
            tls   On
    ```

    [Fluent Bit Documentation](https://docs.fluentbit.io/manual/pipeline/outputs/elasticsearch)

=== "Kinesis Data Stream"

    ``` yaml title="aws-logging-kinesis-config.yaml" linenums="1" hl_lines="11 12"
    kind: ConfigMap
    apiVersion: v1
    metadata:
      name: aws-logging
      namespace: aws-observability
    data:
      output.conf: |
        [OUTPUT]
            Name kinesis
            Match *
            region region-code
            stream my-stream
    ```

    [Fluent Bit Documentation](https://docs.fluentbit.io/manual/pipeline/outputs/kinesis)
    [Github Documentation](https://github.com/aws/amazon-kinesis-streams-for-fluent-bit)

=== "Kinesis Data Firehose"

    ``` yaml title="aws-logging-firehose-config.yaml" linenums="1" hl_lines="11 12"
    kind: ConfigMap
    apiVersion: v1
    metadata:
      name: aws-logging
      namespace: aws-observability
    data:
      output.conf: |
        [OUTPUT]
            Name  kinesis_firehose
            Match *
            region region-code
            delivery_stream my-stream-firehose
    ```

    [Fluent Bit Documentation](https://docs.fluentbit.io/manual/pipeline/outputs/firehose)
    [Github Documentation](https://github.com/aws/amazon-kinesis-firehose-for-fluent-bit)

### Download the IAM policy permission

=== "CloudWatch Logs"

    === ":simple-linux: Linux"

        ``` bash hl_lines="1 2 3 4"
        curl -LO https://raw.githubusercontent.com/aws-samples/amazon-eks-fluent-logging-examples/mainline/examples/fargate/cloudwatchlogs/permissions.json
        ```

    === ":simple-windows: Windows"

        ``` powershell hl_lines="1 2 3 4"
        curl.exe -LO https://raw.githubusercontent.com/aws-samples/amazon-eks-fluent-logging-examples/mainline/examples/fargate/cloudwatchlogs/permissions.json
        ```

=== "ElasticSearch/OpenSearch"

    === ":simple-linux: Linux"

        ``` bash hl_lines="1 2 3 4"
        curl -LO https://raw.githubusercontent.com/aws-samples/amazon-eks-fluent-logging-examples/mainline/examples/fargate/amazon-elasticsearch/permissions.json
        ```

    === ":simple-windows: Windows"

        ``` powershell hl_lines="1 2 3 4"
        curl.exe -LO https://raw.githubusercontent.com/aws-samples/amazon-eks-fluent-logging-examples/mainline/examples/fargate/amazon-elasticsearch/permissions.json
        ```

=== "Kinesis Data Stream"

    === ":simple-linux: Linux"

        ``` bash hl_lines="1 2 3 4"
        curl -LO https://raw.githubusercontent.com/aws-samples/amazon-eks-fluent-logging-examples/mainline/examples/fargate/kinesis-streams/permissions.json
        ```

    === ":simple-windows: Windows"

        ``` powershell hl_lines="1 2 3 4"
        curl.exe -LO https://raw.githubusercontent.com/aws-samples/amazon-eks-fluent-logging-examples/mainline/examples/fargate/kinesis-streams/permissions.json
        ```

=== "Kinesis Data Firehose"

    === ":simple-linux: Linux"

        ``` bash hl_lines="1 2 3 4"
        curl -LO https://raw.githubusercontent.com/aws-samples/amazon-eks-fluent-logging-examples/mainline/examples/fargate/kinesis-firehose/permissions.json
        ```

    === ":simple-windows: Windows"

        ``` powershell hl_lines="1 2 3 4"
        curl.exe -LO https://raw.githubusercontent.com/aws-samples/amazon-eks-fluent-logging-examples/mainline/examples/fargate/kinesis-firehose/permissions.json
        ```

### Attach the IAM policy to the pod execution role

=== ":simple-linux: Linux"

    ``` bash hl_lines="1 2 3 4"
    POLICY_NAME="<policy name>"
    ROLE_NAME="<role name>"
    PROJECT_NAME="<project name>"
    REGION="<region code>"

    POLICY_ARN=$(aws iam create-policy \
        --policy-name $POLICY_NAME \
        --policy-document file://permissions.json \
        --query 'Policy.Arn' \
        --output text \
        # --tags Key=project,Value=$PROJECT_NAME \  # AWS CLI v2
    )

    aws iam attach-role-policy \
        --policy-arn $POLICY_ARN \
        --role-name $ROLE_NAME
    ```

=== ":simple-windows: Windows"

    ``` powershell hl_lines="1 2 3 4"
    $POLICY_NAME="<policy name>"
    $ROLE_NAME="<role name>"
    $PROJECT_NAME="<project name>"
    $REGION="<region code>"

    $POLICY_ARN = aws iam create-policy `
        --policy-name $POLICY_NAME `
        --policy-document file://permissions.json `
        --query 'Policy.Arn' `
        --output text `
        --tags Key=project,Value=$PROJECT_NAME

    aws iam attach-role-policy `
        --policy-arn $POLICY_ARN `
        --role-name $ROLE_NAME
    ```

[AWS Documentation](https://docs.aws.amazon.com/eks/latest/userguide/fargate-logging.html#fargate-logging-log-router-configuration)

[Github](https://github.com/aws/amazon-kinesis-streams-for-fluent-bit)

### (Optional) Add KMS encryption policy to pod execution IAM role

=== "JSON file"
    ``` json title="kinesis-kms-permissions.json" linenums="1"
    {
        "Version": "2012-10-17",
        "Statement": [
            {
                "Effect": "Allow",
                "Action": [
                    "kms:GenerateDataKey"
                ],
                "Resource": "arn:aws:kms:<region code>:<account id>:key/<kms key id>"
            }
        ]
    }
    ```

=== "Using command"
    ``` shell
    cat << EOF > kinesis-kms-permissions.json
    {
        "Version": "2012-10-17",
        "Statement": [
            {
                "Effect": "Allow",
                "Action": [
                    "kms:GenerateDataKey"
                ],
                "Resource": "arn:aws:kms:<region code>:<account id>:key/<kms key id>"
            }
        ]
    }
    EOF

    aws iam create-policy --policy-name <policy name> --policy-document file://kinesis-kms-permissions.json

    aws iam attach-role-policy \
        --policy-arn arn:aws:iam::<account id>:policy/<policy name> \
        --role-name <fargate pod execution role name>
    ```