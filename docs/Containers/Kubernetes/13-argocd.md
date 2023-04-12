---
title: ArgoCD
description: Kubernetes manifests for deploying ArgoCD resources.
---

# ArgoCD

## Application

``` yaml hl_lines="4 16 17 19 22" linenums="1"
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: <application name>
  namespace: argocd
  # annotations:
  #   argocd-image-updater.argoproj.io/image-list: org/app=<IMAGE_REPOSITORY_URL>/<IMAGE_REPOSITORY_NAME> # ex. 123456789012.dkr.ecr.us-east-1.amazonaws.com/nginx
  #   argocd-image-updater.argoproj.io/org_app.allow-tags: any
  #   argocd-image-updater.argoproj.io/org_app.pull-secret: ext:/scripts/auth1.sh
  #   argocd-image-updater.argoproj.io/org_app.update-strategy: semver
spec:
  project: default
  source:
    kustomize:
      images:
        - <IMAGE_REPOSITORY_URL>/<IMAGE_REPOSITORY_NAME>:<IMAGE_REPOSITRY_TAG>  # ex. 123456789012.dkr.ecr.us-east-1.amazonaws.com/nginx:1.0.0
    repoURL: <GIT_REPOSITORY_URL> # ex. https://github.com/argoproj/argocd-example-apps.git
    targetRevision: HEAD          # HEAD or git branch name
    path: <K8S_MANIFEST_PATH>     # ex. k8s
  destination:
    server: https://kubernetes.default.svc
    namespace: <DESTINATION NAMESPACE>
  # syncPolicy:
  #   automated:
  #     prune: true
  #     selfHeal: true
```

<!-- ## Project -->

## Repository

``` yaml hl_lines="4 10 11 12" linenums="1"
apiVersion: v1
kind: Secret
metadata:
  name: <secret name>
  namespace: argocd
  labels:
    argocd.argoproj.io/secret-type: repository
stringData:
  type: git
  url: <repo url> # ex. https://github.com/argoproj/private-repo
  username: <repo username>
  password: <repo password>
```