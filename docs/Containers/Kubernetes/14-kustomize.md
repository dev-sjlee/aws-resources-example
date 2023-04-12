---
title: Kustomize
description: Kubernetes manifests for deploying kustomized resources.
---

# Kustomize

``` yaml title="kustomization.yaml"
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
  - deployment.yaml
  - service.yaml
  - ...
images:
  - name: <IMAGE NAME>  # ex. nginx
    # newName: <NEW IMAGE NAME>
    # newTag: 1.23.1
```