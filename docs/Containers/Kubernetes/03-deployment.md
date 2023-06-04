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
  # do not include replicas in the manifests if you want replicas to be controlled by HPA
  # replicas: 2
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      serviceAccountName: nginx
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
              memory: "500Mi"
            limits:
              cpu: "500m"
              memory: "1000Mi"
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
          # volumeMounts:
          #   - name: volume-1
          #     mountPath: "/mnt/volume-1"
          #     readOnly: true
      terminationGracePeriodSeconds: 60
      # volumes:
      #   - name: volume-1
      #     emptyDir: {}
      # tolerations:
      #   - key: "key1"         # taint key
      #     value: "value1"     # taint value
      #     operator: "Equal"
      #     effect: "NoSchedule"
      # nodeSelector:
      #   key: value            # node label key and value
      # topologySpreadConstraints:
      #   - maxSkew: 1
      #     whenUnsatisfiable: DoNotSchedule
      #     topologyKey: topology.kubernetes.io/zone
      #     labelSelector:
      #       matchLabels:
      #         app: nginx
```

[Kubernetes Documentation](https://kubernetes.io/ko/docs/concepts/workloads/controllers/deployment/)

!!! quote "Leaving Room For Imperativeness"

    It may be desired to leave room for some imperativeness/automation, and not have everything defined in your Git manifests. For example, if you want the number of your deployment's replicas to be managed by Horizontal Pod Autoscaler, then you would not want to track replicas in Git.

    [ArgoCD Best Practices](https://argo-cd.readthedocs.io/en/stable/user-guide/best_practices/#leaving-room-for-imperativeness)