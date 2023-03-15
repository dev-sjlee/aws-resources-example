---
title: Using Docker on Kubernetes Pod
description: Using docker on kubernetes pod.
---

# Using Docker on Kubernetes Pod

## Deploy docker pod

``` yaml linenums="1" title="docker-pod.yaml"
apiVersion: v1
kind: ServiceAccount
metadata:
  name: internal-kubectl
---
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: modify-pods
rules:
  - apiGroups: [""]
    resources:
      - pods
    verbs:
      - get
      - list
      - delete
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: modify-pods-to-sa
subjects:
  - kind: ServiceAccount
    name: internal-kubectl
roleRef:
  kind: Role
  name: modify-pods
  apiGroup: rbac.authorization.k8s.io
---
apiVersion: batch/v1
kind: Job
metadata:
  name: kubectl-job
  namespace: default
  labels:
    app: kubectl-job
spec:
  template:
    spec:
      serviceAccountName: internal-kubectl
      containers:
        - name: kubectl
          image: bitnami/kubectl:1.26.2-debian-11-r4
          command:
            - "sh"
            - "-c"
            - "sleep 3600 && kubectl delete pod -n default docker && sleep 10 && exit 0"
      restartPolicy: Never
      # tolerations:
      #   - key: "key1"         # taint key
      #     value: "value1"     # taint value
      #     operator: "Equal"
      #     effect: "NoSchedule"
      # nodeSelector:
      #   key1: value1            # node label key and value
  backoffLimit: 0
  ttlSecondsAfterFinished: 10
---
apiVersion: v1
kind: Pod
metadata:
  name: docker
  namespace: default
  labels:
    app: docker
spec:
  containers:
    - name: docker
      image: docker:23.0.1-dind-alpine3.17
      securityContext:
        privileged: true
  # tolerations:
  #   - key: "key1"         # taint key
  #     value: "value1"     # taint value
  #     operator: "Equal" 
  #     effect: "NoSchedule"
  # nodeSelector:
  #   key1: value1            # node label key and value
```

``` shell
kubectl apply -f docker-pod.yaml
```

## Access docker pod

``` shell
kubectl exec -it docker -n default -- sh
```

!!! note

    You can use docker command after few seconds.

## Delete docker pod

``` shell
kubectl delete -f docker-pod.yaml
```