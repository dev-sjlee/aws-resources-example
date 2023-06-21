# Ingress

[Kubernetes Documentation](https://kubernetes.io/ko/docs/concepts/services-networking/ingress/)

## Using AWS ALB

``` yaml title="ingress.yaml" hl_lines="4 5 20 24 26" linenums="1"
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: nginx
  namespace: nginx
  annotations:
    alb.ingress.kubernetes.io/scheme: internet-facing
    alb.ingress.kubernetes.io/target-type: ip
    alb.ingress.kubernetes.io/load-balancer-name: nginx
    alb.ingress.kubernetes.io/security-groups: <security group ids>
    alb.ingress.kubernetes.io/healthcheck-path: <healthcheck path>
    alb.ingress.kubernetes.io/tags: <tags>  # Environment=dev,Team=test
    alb.ingress.kubernetes.io/load-balancer-attributes: access_logs.s3.enabled=true,access_logs.s3.bucket=<access log bucket>,access_logs.s3.prefix=<access log prefix>
    # Please check documentations for other annotations.
spec:
  ingressClassName: alb
  rules:
    - http:
        paths:
          - path: /
            pathType: Prefix  # Exact
            backend:
              service:
                name: nginx
                port:
                  number: 80
```

[Go to guide](/aws-resources-example/Containers/EKS/05-using-elb-on-eks/#create-alb-using-ingress)