# Pod Security Group

## Add policy to cluster IAM role

``` shell hl_lines="3"
aws iam attach-role-policy \
    --policy-arn arn:aws:iam::aws:policy/AmazonEKSVPCResourceController \
    --role-name <cluster role>
```

[AWS Documentation](https://docs.aws.amazon.com/ko_kr/eks/latest/userguide/security-groups-for-pods.html#security-groups-pods-deployment)

## Update `aws-node` daemonset environment variable

> Please use this command before starting node groups or fargates.

``` shell
kubectl set env daemonset aws-node -n kube-system ENABLE_POD_ENI=true

kubectl patch daemonset aws-node \
  -n kube-system \
  -p '{"spec": {"template": {"spec": {"initContainers": [{"env":[{"name":"DISABLE_TCP_EARLY_DEMUX","value":"true"}],"name":"aws-vpc-cni-init"}]}}}}'
```

[AWS Documentation](https://docs.aws.amazon.com/ko_kr/eks/latest/userguide/security-groups-for-pods.html#security-groups-pods-deployment)

## Create `SecurityGroupPolicy`

``` yaml title="security-group-policy.yaml" hl_lines="4 5"
apiVersion: vpcresources.k8s.aws/v1beta1
kind: SecurityGroupPolicy
metadata:
  name: <security group policy name>
  namespace: <namespace>
spec:
  podSelector: 
    matchLabels:
      role: my-role # labels
  securityGroups:
    groupIds:
      - sg-abc123   # security groups id
```

``` shell
kubectl apply -f security-group-policy.yaml
```