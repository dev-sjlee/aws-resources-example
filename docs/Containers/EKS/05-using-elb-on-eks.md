# Using ELB on EKS

## Create AWS Load Balancer Controller IAM role

``` shell hl_lines="4 8 11 12"
curl -o aws-load-balancer-controller-iam-policy.json https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.4.2/docs/install/iam_policy.json

aws iam create-policy \
    --policy-name <policy name> \
    --policy-document file://aws-load-balancer-controller-iam-policy.json

eksctl create iamserviceaccount \
    --cluster=<cluster name> \
    --namespace=kube-system \
    --name=aws-load-balancer-controller \
    --role-name "<role name>" \
    --attach-policy-arn=arn:aws:iam::<account id>:policy/<policy name> \
    --override-existing-serviceaccounts \
    --approve
```

[AWS Documentation](https://docs.aws.amazon.com/eks/latest/userguide/aws-load-balancer-controller.html)

## Install AWS Load Balancer Controller using `helm`

``` shell hl_lines="6 9 10"
helm repo add eks https://aws.github.io/eks-charts
helm repo update

helm install aws-load-balancer-controller eks/aws-load-balancer-controller \
    -n kube-system \
    --set clusterName=<cluster name> \
    --set serviceAccount.create=false \
    --set serviceAccount.name=aws-load-balancer-controller \
    --set region=<region code> \
    --set vpcId=<vpc id>

kubectl get deployment aws-load-balancer-controller \
    -n kube-system \
    -w
```

[AWS Documentation](https://docs.aws.amazon.com/eks/latest/userguide/aws-load-balancer-controller.html)

## Create ALB using Ingress

``` yaml title="ingress.yaml" hl_lines="4 5 18 20" linenums="1"
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  namespace: <namespace>
  name: <ingress name>
  annotations:
    kubernetes.io/ingress.class: alb
    alb.ingress.kubernetes.io/scheme: internet-facing
    alb.ingress.kubernetes.io/target-type: ip
    # Please check documentations for other annotations.
spec:
  rules:
    - http:
        paths:
          - path: /
            pathType: Prefix
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