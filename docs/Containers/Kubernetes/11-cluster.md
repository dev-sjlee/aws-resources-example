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
  version: "1.24" # default is "1.24"
  # tags:
  #   <key1>: <value1>
  #   <key2>: <value2>

kubernetesNetworkConfig:
  ipFamily: IPv4  # IPv6

iam:
  serviceRoleARN: <eks cluster role arn>
  withOIDC: true
  serviceAccounts:
    - metadata: # aws load balancer controller
        name: aws-load-balancer-controller
        namespace: kube-system
      wellKnownPolicies:
        awsLoadBalancerController: true
      roleName: <eks elb controller role name>
      tags:
        Name: <eks elb controller role name>

    - metadata: # cluster autoscaler
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
    groups:
      - system:masters
    username: <identity mapping user name>

vpc:
  id: "<vpc id>"  # ex. vpc-0dd338ecf29863c55
  securityGroup: "<security group id>"  # ex. sg-074e7b913be56b86d
  subnets:
    private:
      <az code>:  # ex. eu-north-1a
        id: "<subnet id>" # ex. subnet-0b2512f8c6ae9bf30
      <az code>:  # ex. eu-north-1b
        id: "<subnet id>" # ex. subnet-08cb9a2ed60394ce3
      <az code>:  # ex. eu-north-1c
        id: "<subnet id>" # ex. subnet-00f71956cdec8f1dc
  sharedNodeSecurityGroup: "<general nodegroup security group>" # ex. sg-013f261f9fb5855f7
  manageSharedNodeSecurityGroupRules: false
  # clusterEndpoints:
  #   publicAccess: true
  #   privateAccess: true
  # publicAccessCIDRs:
  #   - 0.0.0.0/0 # or my ip

addons:
  - name: vpc-cni
  - name: coredns
  - name: kube-proxy

privateCluster:
  enabled: true
  skipEndpointCreation: true

managedNodeGroups:
  - name: <node group name> # ex. ng-1
    amiFamily: AmazonLinux2 # Bottlerocket
    instanceType: m5.large
    subnets:
      - "<subnet id>"
      - "<subnet id>"
      - "<subnet id>"
    desiredCapacity: 3
    minSize: 3
    maxSize: 6
    # labels:
    #   key1: value1
    privateNetworking: true
    # tags:
    #   key1: value1
    # taints:
    #   - key:
    #     value:
    #     effect:
    launchTemplate:
      id: <launch template id>
    iam:
      instanceRoleARN: "<node group role arn>"

cloudWatch:
    clusterLogging:
        enableTypes: ["api", "audit", "authenticator", "controllerManager", "scheduler"]

secretsEncryption:
  keyARN: "<kms key arn>" # ex. arn:aws:kms:us-east-1:123456789012:key/12345678-abcd-efgh-ijkl-123456789012
```