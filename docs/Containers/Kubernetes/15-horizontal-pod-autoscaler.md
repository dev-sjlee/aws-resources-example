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
  name: <hpa name>
  namespace: <hpa namespace>
  labels:
    app: <hpa app name>
spec:
  minReplicas: <min replica number (ex. 1)>
  maxReplicas: <max replica number (ex. 10)>
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: <cpu utilization percentage number (ex. 50)>
  scaleTargetRef:
    apiVersion: <target object api version (ex. apps/v1)>
    kind: <target object type (ex. Deployment)>
    name: <target object name>
```

## `autoscaling/v1`

``` yaml title="hpa.yaml"
apiVersion: autoscaling/v1
kind: HorizontalPodAutoscaler
metadata:
  name: <hpa name>
  namespace: <hpa namespace>
  labels:
    app: <hpa app name>
spec:
  minReplicas: <min replica number (ex. 1)>
  maxReplicas: <max replica number (ex. 10)>
  targetCPUUtilizationPercentage: <cpu utilization percentage number (ex. 50)>
  scaleTargetRef:
    apiVersion: <target object api version (ex. apps/v1)>
    kind: <target object type (ex. Deployment)>
    name: <target object name>
```