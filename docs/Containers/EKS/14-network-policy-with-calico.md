# Network Policy with Calico

## Install Calico

``` shell
helm repo add projectcalico https://docs.projectcalico.org/charts
helm repo update                         
helm install calico projectcalico/tigera-operator --version v3.25.0
kubectl get all -n calico-system
```

[AWS Documentation](https://docs.aws.amazon.com/eks/latest/userguide/calico.html#calico-install)

## Create Network Policy

``` yaml title="networkpolicy.yaml"
# Allow all ingress traffic except traffic from app=app1 pod.
apiVersion: crd.projectcalico.org/v1
kind: GlobalNetworkPolicy
metadata:
  name: <network policy name>
  namespace: <namespace>
spec:
  selector: app == 'app1'
  ingress:
  - action: Deny
    protocol: TCP
    source:
      selector: app == 'app2'
  - action: Allow
    protocol: TCP
  egress:
  - action: Allow
```

``` yaml title="networkpolicy.yaml"
kind: NetworkPolicy
apiVersion: networking.k8s.io/v1
metadata:
  name: <network policy name>
  namespace: <namespace>
spec:
  podSelector:
    matchLabels:
      app: app1 # app-label
  types:
    - ingress
    - egress
  ingress:
    - from:
        - ipBlock:
            cidr: 10.0.1.0/24 # allow from elb
        - ipBlock:
            cidr: 10.0.2.0/24 # allow from elb
        - podSelector:
            matchLabels:
              app: app2      # allow from pod
      ports:
        - protocol: TCP
          port: 8080 # the port which should be Internet-accessible
  egress: {}  # egress any open
```