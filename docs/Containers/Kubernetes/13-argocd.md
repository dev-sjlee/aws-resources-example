# ArgoCD

## Application

``` yaml hl_lines="" linenums="1"
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: <application name>
  namespace: argocd
spec:
  project: default
  source:
    repoURL: <repo url> # ex. https://github.com/argoproj/argocd-example-apps.git
    targetRevision: HEAD
    path: <k8s manifest path>   # ex. guestbook
  destination:
    server: https://kubernetes.default.svc
    namespace: <destination namespace>
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