# Network Policy with Calico

## Install Calico

``` shell
helm repo add projectcalico https://docs.projectcalico.org/charts
helm repo update                         
helm install calico projectcalico/tigera-operator --version v3.21.4
kubectl get all -n tigera-operator
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