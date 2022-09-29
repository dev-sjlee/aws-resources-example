# Deployment

``` yaml title="deployment.yaml"
apiVersion: apps/v1
kind: Deployment
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
          resources:
            requests:
              cpu: "250m"
            limits:
              cpu: "500m"
          readinessProbe:
            httpGet:
              path: /health
              port: 80
            initialDelaySeconds: 30
          lifecycle:
            preStop:
              exec:
                command: ["/bin/sh", "-c", "sleep 60"]
          terminationGracePeriodSeconds: 120
```

[Kubernetes Documentation](https://kubernetes.io/ko/docs/concepts/workloads/controllers/deployment/)