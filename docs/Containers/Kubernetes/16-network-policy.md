---
title: Network Policy
description: Kubernetes manifests for deploying Network Policy.
---

# Network Policy

## Calico Network Policy

``` yaml title="networkpolicy.yaml"
kind: NetworkPolicy
apiVersion: networking.k8s.io/v1
metadata:
  name: nginx
  namespace: nginx
spec:
  podSelector:
    matchLabels:
      app: nginx # app-label
  policyTypes:
    - Ingress
    - Egress
  ingress:
    - from:
        - ipBlock:
            cidr: 10.0.1.0/24 # allow from elb
        - ipBlock:
            cidr: 10.0.2.0/24 # allow from elb
        - podSelector:
            matchLabels:
              app: apache      # allow from pod
      ports:
        - protocol: TCP
          port: 8080 # the port which should be Internet-accessible
  egress: {}  # egress any open
```

## Calico Global Network Policy

``` yaml title="globalnetworkpolicy.yaml"
# Allow all ingress traffic except traffic from app=nginx pod.
apiVersion: crd.projectcalico.org/v1
kind: GlobalNetworkPolicy
metadata:
  name: nginx
spec:
  selector: app == 'nginx'
  ingress:
  - action: Deny
    protocol: TCP
    source:
      selector: app == 'apache'
  - action: Allow
    protocol: TCP
  egress:
  - action: Allow
```