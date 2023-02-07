# Using ELB on EKS

## Create AWS Load Balancer Controller IAM role

``` shell hl_lines="1 2 3 4 5"
CLUSTER_NAME="<cluster name>"
POLICY_NAME="<policy name>"
ROLE_NAME="<role name>"
PROJECT_NAME="<project name>"
REGION="<region>"

curl -o aws-load-balancer-controller-iam-policy.json https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.4.2/docs/install/iam_policy.json

POLICY_ARN=$(aws iam create-policy \
    --policy-name $POLICY_NAME \
    --policy-document file://aws-load-balancer-controller-iam-policy.json \
    --tags Key=project,Value=$PROJECT_NAME \
| jq -r '.Policy.Arn')

eksctl create iamserviceaccount \
    --cluster=$CLUSTER_NAME \
    --namespace=kube-system \
    --name=aws-load-balancer-controller \
    --role-name "$ROLE_NAME" \
    --attach-policy-arn=$POLICY_ARN \
    --tags project=$PROJECT_NAME \
    --region $REGION \
    --override-existing-serviceaccounts \
    --approve
```

[AWS Documentation](https://docs.aws.amazon.com/eks/latest/userguide/aws-load-balancer-controller.html)

## Install AWS Load Balancer Controller using `helm`

``` shell hl_lines="1 2 3"
VPC_ID="<vpc id>"
CLUSTER_NAME="<cluster name>"
REGION="<region>"

helm repo add eks https://aws.github.io/eks-charts
helm repo update

helm install aws-load-balancer-controller eks/aws-load-balancer-controller \
    -n kube-system \
    --set clusterName=$CLUSTER_NAME \
    --set serviceAccount.create=false \
    --set serviceAccount.name=aws-load-balancer-controller \
    --set region=$REGION \
    --set vpcId=$VPC_ID

kubectl get deployment aws-load-balancer-controller \
    -n kube-system \
    -w
```

[AWS Documentation](https://docs.aws.amazon.com/eks/latest/userguide/aws-load-balancer-controller.html)

## Create ALB using Ingress

``` yaml title="ingress.yaml" hl_lines="4 5 20 24 26" linenums="1"
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  namespace: <namespace>
  name: <ingress name>
  annotations:
    kubernetes.io/ingress.class: alb
    alb.ingress.kubernetes.io/scheme: internet-facing
    alb.ingress.kubernetes.io/target-type: ip
    alb.ingress.kubernetes.io/load-balancer-name: <load balancer name>
    alb.ingress.kubernetes.io/security-groups: <security group ids>
    alb.ingress.kubernetes.io/healthcheck-path: <healthcheck path>
    alb.ingress.kubernetes.io/tags: <tags>  # Environment=dev,Team=test
    alb.ingress.kubernetes.io/load-balancer-attributes: access_logs.s3.enabled=true,access_logs.s3.bucket=<access log bucket>,access_logs.s3.prefix=<access log prefix>
    # Please check documentations for other annotations.
spec:
  rules:
    - http:
        paths:
          - path: /
            pathType: Prefix  # Exact
            backend:
              service:
                name: <service name>
                port:
                  number: <port number> # like 80
```

[AWS Documentation](https://docs.aws.amazon.com/eks/latest/userguide/alb-ingress.html)

[AWS Load Balancer Controller Documentation](https://kubernetes-sigs.github.io/aws-load-balancer-controller/v2.4/)

## Create NLB using Service (type: `LoadBalancer`)

``` yaml title="service.yaml" hl_lines="4 5 16 17 18 20" linenums="1"
apiVersion: v1
kind: Service
metadata:
  namespace: <namespace>
  name: <ingress name>
  annotations:
    service.beta.kubernetes.io/aws-load-balancer-type: external
    service.beta.kubernetes.io/aws-load-balancer-nlb-target-type: ip
    service.beta.kubernetes.io/aws-load-balancer-scheme: internet-facing
    # Please check documentations for other annotations.
spec:
  type: LoadBalancer
  selector:
    app: <app label>
  ports:
    - port: <port number> # like 80
      targetPort: <port number> # like 80
      protocol: <protocol> # like TCP
```

[AWS Documentation](https://docs.aws.amazon.com/eks/latest/userguide/network-load-balancing.html)

[AWS Load Balancer Controller Documentation](https://kubernetes-sigs.github.io/aws-load-balancer-controller/v2.4/)