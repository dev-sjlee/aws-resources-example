# Horizontal Pod Autoscaling

## Install the Kubernetes Metrics Server

``` shell
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml

kubectl get deployment metrics-server -n kube-system
```

[AWS Documentation](https://docs.aws.amazon.com/eks/latest/userguide/metrics-server.html)

## Using HPA

``` yaml title="hpa.yaml" hl_lines="4 5 7 9 10 14 17 18 19" linenums="1"
apiVersion: autoscaling/v2beta2
kind: HorizontalPodAutoscaler
metadata:
  name: <hpa name>
  namespace: <hpa namespace>
  labels:
    app: <hpa app name>
spec:
  minReplicas: <min replica number (ex. 1)>
  maxReplicas: <max replica number (ex. 10)>
  metrics:
    - resources:
        name: cpu
        targetAverageUtilization: <cpu utilization percentage number (ex. 50)>
      type: Resource
  scaleTargetRef:
    apiVersion: <target object api version (ex. apps/v1)>
    kind: <target object type (ex. Deployment)>
    name: <target object name>
```

[AWS Documentation](https://docs.aws.amazon.com/eks/latest/userguide/horizontal-pod-autoscaler.html)

[Blog](https://medium.com/dtevangelist/k8s-kubernetes%EC%9D%98-hpa%EB%A5%BC-%ED%99%9C%EC%9A%A9%ED%95%9C-%EC%98%A4%ED%86%A0%EC%8A%A4%EC%BC%80%EC%9D%BC%EB%A7%81-auto-scaling-2fc6aca61c26)