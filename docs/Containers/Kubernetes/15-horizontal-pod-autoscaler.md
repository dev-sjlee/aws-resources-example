---
title: Horizontal Pod Autoscaler
description: Kubernetes manifests for deploying horizontal pod autoscaler resources.
---

# Horizontal Pod Autoscaler

## `autoscaling/v2`

``` yaml title="hpa.yaml"
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: nginx
  namespace: nginx
  labels:
    app: nginx
spec:
  minReplicas: 2
  maxReplicas: 10
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 40
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: nginx
```

## `autoscaling/v1`

``` yaml title="hpa.yaml"
apiVersion: autoscaling/v1
kind: HorizontalPodAutoscaler
metadata:
  name: nginx
  namespace: nginx
  labels:
    app: nginx
spec:
  minReplicas: 2
  maxReplicas: 10
  targetCPUUtilizationPercentage: 40
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: nginx
```