# Using External DNS

## Create the service account for External DNS

=== ":simple-linux: Linux"

    ``` bash hl_lines="1 2 3 4 5"
    CLUSTER_NAME="<cluster name>"
    POLICY_NAME="<policy name>"
    ROLE_NAME="<role name>"
    PROJECT_NAME="<project name>"
    REGION="<region>"

    curl -LO https://raw.githubusercontent.com/marcus16-kang/aws-resources-example/main/scripts/eks/external-dns-iam-policy.json

    POLICY_ARN=$(aws iam create-policy \
        --policy-name $POLICY_NAME \
        --policy-document file://external-dns-iam-policy.json \
        --query 'Policy.Arn' \
        --output text \
        #  --tags Key=project,Value=$PROJECT_NAME \ # AWS CLI v2
    )
    
    eksctl create iamserviceaccount \
        --name external-dns \
        --namespace external-dns \
        --cluster $CLUSTER_NAME \
        --attach-policy-arn $POLICY_ARN \
        --role-name $ROLE_NAME \
        --tags project=$PROJECT_NAME \
        --region $REGION \
        --override-existing-serviceaccounts \
        --approve
    ```

=== ":simple-windows: Windows"

    ``` powershell hl_lines="1 2 3 4 5"
    $CLUSTER_NAME="<cluster name>"
    $POLICY_NAME="<policy name>"
    $ROLE_NAME="<role name>"
    $PROJECT_NAME="<project name>"
    $REGION="<region>"

    curl.exe -LO https://raw.githubusercontent.com/marcus16-kang/aws-resources-example/main/scripts/eks/external-dns-iam-policy.json

    $POLICY_ARN = aws iam create-policy `
        --policy-name $POLICY_NAME `
        --policy-document file://external-dns-iam-policy.json `
        --query 'Policy.Arn' `
        --output text `
        --tags Key=project,Value=$PROJECT_NAME
    
    eksctl create iamserviceaccount `
        --name external-dns `
        --namespace external-dns `
        --cluster $CLUSTER_NAME `
        --attach-policy-arn $POLICY_ARN `
        --role-name $ROLE_NAME `
        --tags project=$PROJECT_NAME `
        --region $REGION `
        --override-existing-serviceaccounts `
        --approve
    ```

[AWS Documentation](https://aws.amazon.com/ko/premiumsupport/knowledge-center/eks-set-up-externaldns/)

[Blog 1](https://velog.io/@ironkey/AWS-EKS%EC%97%90%EC%84%9C-%EB%8F%99%EC%9E%91%EC%A4%91%EC%9D%B8-%EC%BB%A8%ED%85%8C%EC%9D%B4%EB%84%88%EC%97%90-External-DNS%EB%A1%9C-%EB%8F%84%EB%A9%94%EC%9D%B8-%EC%97%B0%EA%B2%B0%ED%95%98%EA%B8%B0#external-dns-1)
[Blog 2](https://nyyang.tistory.com/111)

## Install External DNS using `helm`

=== ":simple-linux: Linux"

    ``` bash hl_lines="1 2"
    DOMAIN_NAME="<domain name (ex. my-org.com)>"
    ZONE_TYPE="<zone type (public or private)>"

    helm repo add external-dns https://kubernetes-sigs.github.io/external-dns/

    helm upgrade --install external-dns external-dns/external-dns \
        --namespace external-dns \
        --set serviceAccount.create=false \
        --set serviceAccount.name=external-dns \
        --set policy=sync \
        --set registry=txt \
        --set txtOwnerId=my-hostedzone-identifier \
        --set domainFilters[0]=$DOMAIN_NAME \
        --set provider=aws \
        --set extraArgs[0]="--aws-zone-type=$ZONE_TYPE"
    ```

=== ":simple-windows: Windows"

    ``` powershell hl_lines="1 2"
    $DOMAIN_NAME="<domain name (ex. my-org.com)>"
    $ZONE_TYPE="<zone type (public or private)>"

    helm repo add external-dns https://kubernetes-sigs.github.io/external-dns/

    helm upgrade --install external-dns external-dns/external-dns `
        --namespace external-dns `
        --set serviceAccount.create=false `
        --set serviceAccount.name=external-dns `
        --set policy=sync `
        --set registry=txt `
        --set txtOwnerId=my-hostedzone-identifier `
        --set domainFilters[0]=$DOMAIN_NAME `
        --set provider=aws `
        --set extraArgs[0]="--aws-zone-type=$ZONE_TYPE"
    ```

[External DNS Helm Chart Documentation](https://github.com/kubernetes-sigs/external-dns/tree/master/charts/external-dns)

## Install External DNS using YAML files

### Create the namespace for External DNS

=== "YAML file"
    ``` yaml title="external-dns-namespace.yaml"
    apiVersion: v1
    kind: Namespace
    metadata:
      name: external-dns
    ```

=== "Using command"
    ```shell
    cat << EOF > external-dns-namespace.yaml
    apiVersion: v1
    kind: Namespace
    metadata:
      name: external-dns
    EOF

    kubectl apply -f external-dns-namespace.yaml
    ```

[AWS Documentation](https://aws.amazon.com/ko/premiumsupport/knowledge-center/eks-set-up-externaldns/)

[Blog 1](https://velog.io/@ironkey/AWS-EKS%EC%97%90%EC%84%9C-%EB%8F%99%EC%9E%91%EC%A4%91%EC%9D%B8-%EC%BB%A8%ED%85%8C%EC%9D%B4%EB%84%88%EC%97%90-External-DNS%EB%A1%9C-%EB%8F%84%EB%A9%94%EC%9D%B8-%EC%97%B0%EA%B2%B0%ED%95%98%EA%B8%B0#external-dns-1)
[Blog 2](https://nyyang.tistory.com/111)

### Deploy External DNS

``` yaml title="external-dns-deployment.yaml" linenums="1" hl_lines="54 56 57"
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: external-dns
  namespace: external-dns
rules:
- apiGroups: [""]
  resources: ["services","endpoints","pods"]
  verbs: ["get","watch","list"]
- apiGroups: ["extensions","networking.k8s.io"]
  resources: ["ingresses"]
  verbs: ["get","watch","list"]
- apiGroups: [""]
  resources: ["nodes"]
  verbs: ["list","watch"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: external-dns-viewer
  namespace: external-dns
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: external-dns
subjects:
- kind: ServiceAccount
  name: external-dns
  namespace: external-dns
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: external-dns
  namespace: external-dns
spec:
  strategy:
    type: Recreate
  selector:
    matchLabels:
      app: external-dns
  template:
    metadata:
      labels:
        app: external-dns
    spec:
      serviceAccountName: external-dns
      containers:
      - name: external-dns
        image: k8s.gcr.io/external-dns/external-dns:v0.7.6
        args:
        - --source=service
        - --source=ingress
        - --domain-filter=<domain name (ex. external-dns-test.my-org.com)> # will make ExternalDNS see only the hosted zones matching provided domain, omit to process all available hosted zones
        - --provider=aws
        - --policy=<dns policy (ex. upsert-only)> # would prevent ExternalDNS from deleting any records, omit to enable full synchronization
        - --aws-zone-type=<hosted zone type (ex. public)> # only look at public hosted zones (valid values are public, private or no value for both)
        - --registry=txt
        - --txt-owner-id=my-hostedzone-identifier
      securityContext:
        fsGroup: 65534 # For ExternalDNS to be able to read Kubernetes and AWS token files
```

### External DNS policy references

- `upsert-only` : CREATE, UPDATE
- `sync` : CREATE, UPDATE, DELETE
- `create-only` : CREATE

[AWS Documentation](https://aws.amazon.com/ko/premiumsupport/knowledge-center/eks-set-up-externaldns/)

[External DNS for AWS Documentation](https://kubernetes-sigs.github.io/external-dns/v0.12.2/tutorials/aws/)

[Blog 1](https://velog.io/@ironkey/AWS-EKS%EC%97%90%EC%84%9C-%EB%8F%99%EC%9E%91%EC%A4%91%EC%9D%B8-%EC%BB%A8%ED%85%8C%EC%9D%B4%EB%84%88%EC%97%90-External-DNS%EB%A1%9C-%EB%8F%84%EB%A9%94%EC%9D%B8-%EC%97%B0%EA%B2%B0%ED%95%98%EA%B8%B0#external-dns-1)
[Blog 2](https://nyyang.tistory.com/111)

## Using External DNS with ALB (in Ingress)

``` yaml title="ingress.yaml" hl_lines="8" linenums="1"
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  namespace: <namespace>
  name: <ingress name>
  annotations:
    kubernetes.io/ingress.class: alb
    external-dns.alpha.kubernetes.io/hostname: <hostname (ex. echoserver.mycluster.example.org)>
    alb.ingress.kubernetes.io/scheme: internet-facing
    alb.ingress.kubernetes.io/target-type: ip
spec:
  rules:
    - http:
        path: /
        pathType: Prefix
        backend:
          service:
            name: <service name>
            port:
              number: <port number> # like 80
```

### Dualstack ALB

``` yaml title="ingress.yaml" hl_lines="9 13" linenums="1"
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  namespace: <namespace>
  name: <ingress name>
  annotations:
    kubernetes.io/ingress.class: alb
    alb.ingress.kubernetes.io/scheme: internet-facing
    alb.ingress.kubernetes.io/ip-address-type: dualstack
    alb.ingress.kubernetes.io/target-type: ip
spec:
  rules:
    - host: <hostname (ex. echoserver.example.org)>
      http:
        path: /
        pathType: Prefix
        backend:
          service:
            name: <service name>
            port:
              number: <port number> # like 80
```

[External DNS Documentation](https://kubernetes-sigs.github.io/external-dns/v0.12.2/tutorials/alb-ingress/)

## Using External DNS with NLB

``` yaml title="service.yaml" hl_lines="7" linenums="1"
apiVersion: v1
kind: Service
metadata:
  namespace: <namespace>
  name: <ingress name>
  annotations:
    external-dns.alpha.kubernetes.io/hostname: <hostname (ex. echoserver.example.org)>
    service.beta.kubernetes.io/aws-load-balancer-type: external
    service.beta.kubernetes.io/aws-load-balancer-nlb-target-type: ip
    service.beta.kubernetes.io/aws-load-balancer-scheme: internet-facing
spec:
  type: LoadBalancer
  selector:
    app: <app label>
  ports:
    - port: <port number> # like 80
      targetPort: <port number> # like 80
      protocol: <protocol> # like TCP
```

[External DNS Documentation](https://kubernetes-sigs.github.io/external-dns/v0.12.2/tutorials/aws/)

## Using External DNS with NodePort Service

``` yaml title="service.yaml"
apiVersion: v1
kind: Service
metadata:
  name: <name>
  namespace: <namespace>
  annotations:
    external-dns.alpha.kubernetes.io/hostname: <hostname (ex. echoserver.example.org)>
spec:
  selector:
    app: <app label>
  type: NodePort
  ports:
    - port: <port number> # like 80
      targetPort: <port number> # like 80
      protocol: <protocol> # like TCP
```

!!! warning

    If you use `ClusterIP`, you **CANNOT** use External DNS.