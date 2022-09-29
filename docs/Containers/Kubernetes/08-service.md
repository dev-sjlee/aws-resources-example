# Service

``` yaml title="nodeport_service.yaml"
apiVersion: v1
kind: Service
metadata:
  name: nginx
  namespace: nginx
  labels:
    app: nginx
spec:
  selector:
    app: nginx
  type: NodePort
  ports:
    - protocol: TCP
      port: 80
      targetPort : 80 # container port
```

[Kubernetes Documentation](https://kubernetes.io/ko/docs/concepts/services-networking/service/)

## Using AWS NLB

[Go to guide](/aws-resources-example/Containers/EKS/05-using-elb-on-eks/#create-nlb-using-service-type-loadbalancer)