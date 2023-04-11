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
  replicas: 2
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      serviceAccountName: nginx-sa
      containers:
        - name: nginx
          image: nginx:1.14.2
          ports:
            - containerPort: 80
          # env:
          #   - name: TEST
          #     value: "Hello, World!"
          resources:
            requests:
              cpu: "250m"
            limits:
              cpu: "500m"
          # command:  # override Dockerfile `ENTRYPOINT`
          #   - ""
          #   - ""
          # args:     # override Dockerfile `CMD`
          #   - ""
          #   - ""
          readinessProbe:
            httpGet:
              path: /health
              port: 80
            initialDelaySeconds: 15
          livenessProbe:
            httpGet:
              path: /health
              port: 80
            initialDelaySeconds: 30
          lifecycle:
            preStop:
              exec:
                command: ["/bin/sh", "-c", "sleep 60"]
        terminationGracePeriodSeconds: 60
      # tolerations:
      #   - key: "key1"         # taint key
      #     value: "value1"     # taint value
      #     operator: "Equal"
      #     effect: "NoSchedule"
      # nodeSelector:
      #   key: value            # node label key and value
```

[Kubernetes Documentation](https://kubernetes.io/ko/docs/concepts/workloads/controllers/deployment/)