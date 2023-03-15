---
title: Install Descheduler
description: Install Descheduler using helm.
---

# Install Descheduler

## Using `helm`

``` shell
helm repo add descheduler https://kubernetes-sigs.github.io/descheduler/
helm install descheduler --namespace kube-system descheduler/descheduler
```

## Default Configuration

``` yaml linenums="1"
apiVersion: v1
data:
  policy.yaml: |
    apiVersion: "descheduler/v1alpha1"
    kind: "DeschedulerPolicy"
    strategies:
      LowNodeUtilization:
        enabled: true
        params:
          nodeResourceUtilizationThresholds:
            targetThresholds:
              cpu: 50
              memory: 50
              pods: 50
            thresholds:
              cpu: 20
              memory: 20
              pods: 20
      RemoveDuplicates:
        enabled: true
      RemovePodsHavingTooManyRestarts:
        enabled: true
        params:
          podsHavingTooManyRestarts:
            includingInitContainers: true
            podRestartThreshold: 100
      RemovePodsViolatingInterPodAntiAffinity:
        enabled: true
      RemovePodsViolatingNodeAffinity:
        enabled: true
        params:
          nodeAffinityType:
          - requiredDuringSchedulingIgnoredDuringExecution
      RemovePodsViolatingNodeTaints:
        enabled: true
      RemovePodsViolatingTopologySpreadConstraint:
        enabled: true
        params:
          includeSoftConstraints: false
kind: ConfigMap
metadata:
  annotations:
    meta.helm.sh/release-name: descheduler
    meta.helm.sh/release-namespace: kube-system
  creationTimestamp: "2023-02-10T00:27:07Z"
  labels:
    app.kubernetes.io/instance: descheduler
    app.kubernetes.io/managed-by: Helm
    app.kubernetes.io/name: descheduler
    app.kubernetes.io/version: 0.26.0
    helm.sh/chart: descheduler-0.26.0
  name: descheduler
  namespace: kube-system
  resourceVersion: "157730"
  uid: 4151cd1e-5cb1-493c-966d-68228d1d0a75
```