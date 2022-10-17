# Cluster

``` shell
eksctl create cluster -f cluster.yaml
```

``` yaml
apiVersion: eksctl.io/v1alpha5
kind: ClusterConfig

metadata:
  name: <cluster name>
  region: <region code>

iam:
  serviceRoleARN: <eks cluster role arn>
  withOIDC: true
  serviceAccounts:
    - metadata: # cluster autoscaler
        name: aws-load-balancer-controller
        namespace: kube-system
      wellKnownPolicies:
        awsLoadBalancerController: true
      roleName: <eks elb controller role name>
      tags:
        Name: <eks elb controller role name>

    - metadata: # aws load balancer controller
        name: cluster-autoscaler
        namespace: kube-system
      wellKnownPolicies:
        autoScaler: true
      roleName: <cluster autoscaler role name>
      tags:
        Name: <cluster autoscaler role name>

    - metadata: # external dns
        name: external-dns
        namespace: kube-system
      wellKnownPolicies:
        externalDNS: true
      roleName: <external dns role name>
      tags:
        Name: <external dns role name>
    
    - metadata: # efs csi controller
        name: efs-csi-controller-sa
        namespace: kube-system
      wellKnownPolicies:
        externalDNS: true
      roleName: <efs csi controller role name>
      tags:
        Name: <efs csi controller role name>

iamIdentityMappings:
  - arn: <custom iam resources arn>

identityProviders:
  - type: oidc

vpc:
  id: "<vpc id>"  # ex. vpc-0dd338ecf29863c55
  subnets:
    private:
      <az code>:  # ex. eu-north-1a
        id: "<subnet id>" # ex. subnet-0b2512f8c6ae9bf30
      <az code>:  # ex. eu-north-1b
        id: "<subnet id>" # ex. subnet-08cb9a2ed60394ce3
      <az code>:  # ex. eu-north-1c
        id: "<subnet id>" # ex. subnet-00f71956cdec8f1dc

nodeGroups:
  - name: <node group name> # ex. ng-1
    instanceType: m5.large
    desiredCapacity: 2

cloudWatch:
    clusterLogging:
        enableTypes: ["api", "audit", "authenticator", "controllerManager", "scheduler"]
```