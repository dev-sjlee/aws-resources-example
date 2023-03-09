# ServiceAccount

## Create usint YAML

``` yaml title="service-account"
apiVersion: v1
kind: ServiceAccount
metadata:
  name: my-service-account
  namespace: default
  annotations:
    eks.amazonaws.com/role-arn: arn:aws:iam::$account_id:role/my-role
```

## Create using `eksctl` to use IRSA

``` shell
eksctl create iamserviceaccount \
    --cluster <CLUSTER_NAME> \
    --name <SERVICE_ACCOUNT_NAME> \
    --namespace <SERVICE_ACCOUNT_NAMESPACE> \
    --attach-policy-arn <POLICY_ARN> \
    --role-name <ROLE_NAME> \
    --tags <TAGS (ex. k1=v1,k2=v2)> \
    --region <REGION_CODE> \
    --approve
```