# Pod

``` yaml title="pod.yaml"
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  namespace: nginx
  labels:
    app: nginx
spec:
  containers:
    - name: nginx
      image: nginx:1.14.2
      ports:
        - containerPort: 80
```

[Kubernetes Documentation](https://kubernetes.io/ko/docs/concepts/workloads/pods/)