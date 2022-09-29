# StatefulSet

``` yaml title="statefulset.yaml"
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: nginx
  namespace: nginx
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
        - name: nginx
          image: nginx:1.14.2
          ports:
            - containerPort: 80
```

[Kubernetes Documentation](https://kubernetes.io/ko/docs/concepts/workloads/controllers/statefulset/)