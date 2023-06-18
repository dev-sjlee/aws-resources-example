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
  serviceAccountName: nginx
  containers:
    - name: nginx
      image: nginx:1.14.2
      ports:
        - containerPort: 80
      command:  # override Dockerfile `ENTRYPOINT`
        - ""
        - ""
      args:     # override Dockerfile `CMD`
        - ""
        - ""
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

[Kubernetes Documentation](https://kubernetes.io/ko/docs/concepts/workloads/pods/)