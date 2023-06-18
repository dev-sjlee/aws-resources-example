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

## Application Set

``` yaml hl_lines="4" linenums="1"
apiVersion: argoproj.io/v1alpha1
kind: ApplicationSet
metadata:
  name: <application name>
  namespace: argocd
spec:
  generators:
  - list:
      elements:
      - cluster: dev
        url: <server url>
      - cluster: prod
        url: <server url>
  template:
    metadata:
      name: '{{cluster}}-nginx'
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

## Cluster

``` yaml hl_lines="4 12 15 18 19" linenums="1"
apiVersion: v1
kind: Secret
metadata:
  name: CLUSTER_NAME
  namespace: argocd
  labels:
    argocd.argoproj.io/secret-type: cluster
type: Opaque
stringData:
  config: |
    {
      "bearerToken":"TOKEN",
      "tlsClientConfig":{
        "insecure":false,
        "caData":"CA_DATA"
      }
    }
  name: CLUSTER_NAME
  server: SERVER_URL
```

## Rollout

``` yaml linenums="1"
apiVersion: argoproj.io/v1alpha1
kind: Rollout
metadata:
  name: nginx
  namespace: nginx
  labels:
    app: nginx
spec:
  # do not include replicas in the manifests if you want replicas to be controlled by HPA
  # replicas: 2
  revisionHistoryLimit: 2
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
          image: nginx:latest
          ports:
            - containerPort: 8080
          # env:
          #   - name: TEST
          #     value: "Hello, World!"
          resources:
            requests:
              cpu: "250m"
              memory: "250Mi"
            limits:
              cpu: "500m"
              memory: "500Mi"
          # command:  # override Dockerfile `ENTRYPOINT`
          #   - ""
          #   - ""
          # args:     # override Dockerfile `CMD`
          #   - ""
          #   - ""
          readinessProbe:
            httpGet:
              path: /health
              port: 8080
            initialDelaySeconds: 2
          livenessProbe:
            httpGet:
              path: /health
              port: 8080
            initialDelaySeconds: 3
          lifecycle:
            preStop:
              exec:
                command: ["/bin/sh", "-c", "sleep 2"]
          # volumeMounts:
          #   - name: volume-1
          #     mountPath: "/mnt/volume-1"
          #     readOnly: true
      terminationGracePeriodSeconds: 3
      # volumes:
      #   - name: volume-1
      #     emptyDir: {}
      # tolerations:
      #   - key: "type"         # taint key
      #     value: "nginx"     # taint value
      #     operator: "Equal"
      #     effect: "NoSchedule"
      # nodeSelector:
      #   type: nginx            # node label key and value
      # topologySpreadConstraints:
      #   - maxSkew: 1
      #     whenUnsatisfiable: DoNotSchedule
      #     topologyKey: topology.kubernetes.io/zone
      #     labelSelector:
      #       matchLabels:
      #         app: nginx
  strategy:
    canary:
      canaryService: nginx-canary
      stableService: nginx-stable
      trafficRouting:
        alb:
          ingress: nginx
          rootService: nginx-root
          servicePort: 8080
      steps:
        - setWeight: 20
        - pause: {duration: 10s}
        - setWeight: 40
        - pause: {duration: 10s}
        - setWeight: 60
        - pause: {duration: 10s}
        - setWeight: 80
        - pause: {duration: 10s}
```

[Argo Rollout Documentation](https://argo-rollouts.readthedocs.io/en/stable/)
[Sample Application](https://github.com/marcus16-kang/kubernetes-example/tree/main/argo-rollout)